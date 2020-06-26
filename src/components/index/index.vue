<template>
  <div>
    <header class="header__content" style="position: relative; overflow: hidden;">
      <Decorate />
      <div class="container">
        <div class="he-row">
          <div class="header_left">
            <div class="logo">
              <Icon type="md-cube" />
            </div>
            <div class="header__info">
              <span>
                <h2>工作台</h2>
                <p>这里将展示你的个人项目，当然也包括协同项目。</p>
              </span>
            </div>
          </div>

          <div class="header__control">
            <RadioGroup v-model="qurey" type="button" @on-change="handleOnChange">
              <Radio label="全部"></Radio>
              <Radio label="我创建的"></Radio>
              <Radio label="已收藏的"></Radio>
            </RadioGroup>
            <Button
              shape="circle"
              icon="ios-power-outline"
              style="margin-left:60px;"
              @click="handleServer"
            ></Button>
            <!-- <Button shape="circle"  icon="ios-power-outline" ></Button> -->
          </div>
        </div>
      </div>
    </header>
    <section>
      <div>
        <transition name="fade">
          <ul class="box" v-if="projectData.length">
            <li v-for="(item,i) in projectData" :key="item.bid+i">
              <div class="icon">
                <Icon
                  type="ios-star-outline"
                  size="26"
                  v-if="item.collect!=='1'"
                  @click="handleStar(item.bid,'1')"
                />
                <Icon type="ios-star" size="26" v-else @click="handleStar(item.bid,'0')" />
              </div>
              <h2 @click="handleHref(item)">{{item.projectName}}</h2>
              <p>
                <Icon type="ios-code-working" size="20" />
                <span>{{item.version}}</span>
              </p>
              <p>
                <Icon type="ios-settings-outline" size="18" />
                <span>{{item.mode=="0"?"静态部署":"自动部署"}}</span>
              </p>
              <p>
                <Icon type="ios-time-outline" size="18" />
                <span>{{item.time}}</span>
              </p>
              <div class="description">{{item.remark}}</div>
              <div class="bottom-list">
                <Tooltip
                  content="复制链接"
                  placement="top"
                  class="border-r-no"
                  v-clipboard:copy="item.href"
                  v-clipboard:success="onCopy"
                  v-clipboard:error="onError"
                >
                  <Icon type="ios-link" size="20" />
                </Tooltip>
                <Tooltip
                  content="删除"
                  placement="top"
                  class="border-r-no"
                  @click.native="handleDelete(item)"
                  v-if="user.bid==item.authorId"
                >
                  <Icon type="ios-trash" size="20" />
                </Tooltip>
                <Tooltip
                  content="部署列表"
                  placement="top"
                  class="border-r-no"
                  @click.native="handleRouter('/tablePage',item.key)"
                >
                  <Icon type="ios-list-box-outline" size="20" />
                </Tooltip>
                <Tooltip
                  content="更新项目"
                  placement="top"
                  @click.native="handleRouter('/addpage',item.key,item)"
                >
                  <Icon type="md-repeat" size="20" />
                  <!-- <Icon type="md-add"/> -->
                </Tooltip>
              </div>
            </li>
          </ul>
        </transition>

        <!-- 无数据时展示的内容 -->
        <div class="placeholder" v-show="!projectData.length">
          <p>ヾ(^∀^)ﾉ</p>
          <p>
            暂无内容哦，快去
            <router-link to="/addpage">创建项目</router-link>吧！
          </p>
        </div>
        <!-- 右下角添加按钮 -->
        <div class="em-add" @click="handleRouter('/addpage')">
          <Icon type="md-add" />
        </div>

        <!-- 删除弹窗 -->
        <Modal v-model="isDelete" width="360" @on-cancel="handleCancel">
          <p slot="header" style="color:#f60;text-align:center">
            <Icon type="ios-information-circle"></Icon>
            <span>系统提示</span>
          </p>
          <div style="text-align:center">
            <p>
              出于某些原因，删除也许会失败。但如果你执意删除，必须知道此操作无法撤消，这将永久删除。
              <br />请输入项目名称以进行确认。
            </p>
            <!-- v-model="keyInput" -->
            <!-- <AutoComplete
              @on-select="handleSearch"
              placeholder="请输入要删除的项目名称..."
              style="width: 98%;margin-top:15px;"
            >
              <Option
                v-for="(item,i) in projectNameArr"
                :value="item.key"
                :key="'g'+i"
              >{{ item.projectName }}</Option>
            </AutoComplete>-->
            <Select v-model="key" style="width: 98%;margin-top:15px;" @on-change="handleSearch">
              <Option
                v-for="(item,i) in projectNameArr"
                :value="item.key"
                :key="item.key+i"
              >{{ item.projectName }}</Option>
            </Select>
          </div>
          <div slot="footer">
            <Button
              type="error"
              size="large"
              long
              :disabled="buttonDisabled"
              @click="handleDelModal"
            >确认删除</Button>
          </div>
        </Modal>
        <!-- 权限控制 -->
        <!-- <Modal v-model="isToLogin" width="360" @on-cancel="handleToLogin">
          <p slot="header" style="color:#f60;text-align:center">
            <Icon type="ios-information-circle"></Icon>
            <span>系统提示</span>
          </p>
          <div style="text-align:center">
            <p>由于您不是注册用户，暂无权执行此操作，请注册登录后操作！</p>
          </div>
          <div slot="footer">
            <Button type="info" size="large" long @click="handleLoginModal">我要去注册</Button>
          </div>
        </Modal>-->
      </div>
    </section>
  </div>
</template>

<script>
import Decorate from "../header/decorate";
export default {
  components: {
    Decorate
  },
  name: "projecrtlist",
  // props: {
  //   msg: String
  // },

  data: () => ({
    projectData: [],
    qurey: "我创建的",
    authorId: "",
    root: "",
    collect: "",
    key: "",
    keyInput: "",
    projectNameArr: [],
    isDelete: false,
    buttonDisabled: true,
    // isToLogin: false,
    user: {}
  }),
  mounted() {
    this.$Message.destroy();
    this.user = this.$store.state.variable.info;
    this.authorId = this.user.bid;
    if (this.user.name === "admin") {
      this.authorId = "";
      this.collect = "";
      this.qurey = "全部";
    }
    this.handleGetData();
    // this.$event.on("input", val => {
    //   this.projectName = val;
    //   this.handleGetData();
    // });
  },
  methods: {
    handleGetData() {
      // this.$Message.destroy();
      let data = {
        // pageNo: this.pageNo,
        // pageSize: this.pageSize
        // pageSize: this.pageSize
        idDeployment: "yes"
      };
      if (this.authorId) {
        data.authorId = this.authorId;
      }
      if (this.collect) {
        data.collect = this.collect;
      }
      if (this.key) {
        data.key = this.key;
      }
      this.$axios
        .get("/api/deploy/edition/get", { params: data })
        .then(res => {
          if (res.data.result) {
            let data = res.data.list;
            data.forEach(item => {
              let href = this.$url;
              if (item.port == "") {
                href = this.$url + item.webUrl + "/index.html";
              } else {
                let url = this.$url.slice(0, this.$url.lastIndexOf(":") + 1);
                href = url + item.port;
              }
              item.href = href;
            });
            this.projectData = data;
            this.projectNameArr = res.data.project;
            this.$store.commit("setProjectTitleArr", res.data.project);
          } else {
            this.$Message["error"]({
              background: true,
              content: "数据请求失败！"
            });
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    handleOnChange(key) {
      // this.noauthorId = false;
      this.authorId = "";
      this.collect = "";
      this.handleCancel();
      // this.$event.emit("inputClear", "");
      switch (key) {
        case "我创建的":
          this.authorId = this.user.bid;
          break;
        case "已收藏的":
          this.collect = "1";
          // this.authorId = this.user.bid;
          break;
        // case "启动服务":
        //   this.handleServer();
        //   break;
        default:
          break;
      }

      this.handleGetData();
    },
    handleServer() {
      this.$axios
        .post(
          "/api/service/operation/open",
          this.$qs.stringify({ port: "all" })
        )
        .then(res => {
          this.$Message.destroy();
          if (res.data.result) {
            // this.$Message["success"]({
            //   background: true,
            //   content: "所有端口服务已重启成功！"
            // });
            this.$Modal.success({
              title: "系统提示",
              content: "所有端口服务已重启成功！"
            });
          } else {
            this.$Message["error"]({
              background: true,
              content: "操作失败！"
            });
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    handleStar(bid, collect) {
      this.$axios
        .post(
          "/api/deploy/edition/update",
          this.$qs.stringify({ bid, collect })
        )
        .then(res => {
          let message = collect == "1" ? "收藏成功！" : "已取消收藏！";
          this.$Message.destroy();
          if (res.data.result) {
            this.$Message["success"]({
              background: true,
              content: message
            });
            this.handleGetData();
          } else {
            this.$Message["error"]({
              background: true,
              content: "操作失败！"
            });
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    handleSearch(val) {
      // this.keyInput = val;
      if (val && this.keyInput === val) {
        this.buttonDisabled = false;
      } else {
        this.buttonDisabled = true;
      }
    },
    handleCancel() {
      this.key = "";
      this.keyInput = "";
      this.buttonDisabled = true;
    },
    handleDelModal() {
      this.$Message.destroy();
      this.$axios
        .post(
          "/api/deploy/edition/delete",
          this.$qs.stringify({
            key: this.keyInput,
            vi: "1",
            root: this.root
          })
        )
        .then(res => {
          if (res.data.result) {
            this.$Message["success"]({
              background: true,
              content: "删除成功！"
            });
            this.isDelete = false;
            this.handleCancel();
            this.handleGetData();
          } else {
            this.$Message["error"]({
              background: true,
              content: "删除失败！"
            });
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    handleDelete(val) {
      // if (this.user.name === "Admin") {
      //   this.isToLogin = true;
      // } else {
      this.isDelete = true;
      this.keyInput = val.key;
      this.root = val.root;
      // }
    },
    handleRouter(path, val, data) {
      // this.$store.commit("setItemData", data);
      this.$router.push({ path, query: { bid: val } });
    },
    handleHref(data) {
      let win = window.open();
      win.opener = null;
      win.location = data.href;
      win.target = "_blank";
    },
    onCopy() {
      this.$Message.destroy();
      this.$Message["success"]({
        background: true,
        content: "链接地址已复制到粘贴板！"
      });
    },
    onError() {
      this.$Message.destroy();
      this.$Message["success"]({
        background: true,
        content: "复制失败！"
      });
    }
    // handleToLogin() {
    //   this.isToLogin = false;
    // },
    // handleLoginModal() {
    //   window.sessionStorage.clear();
    //   this.$store.commit("setUser", {});
    //   this.$router.push({ path: "/login" });
    // }
  }
};
</script>
<style lang="scss" scoped>
@import "./index.scss";
</style>

