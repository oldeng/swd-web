<template>
  <div>
    <header class="header__content" style="position: relative; overflow: hidden;">
      <Decorate />
      <div class="container">
        <div class="he-row">
          <div class="header_left">
            <div class="logo">
              <Icon type="md-person" />
            </div>
            <div class="header__info">
              <span>
                <h2>个人中心</h2>
                <p>个性化设置，修改个人基本信息。</p>
              </span>
            </div>
          </div>
        </div>
      </div>
    </header>
    <section>
      <div class="pro-page">
        <div class="warp">
          <!-- 用户名 -->
          <div v-if="isName">
            <label>用户名：</label>
            <span>{{name}}</span>
            <Button type="dashed" v-if="user.name!=='admin'" @click="handleOnChange('yhmxg')">
              <Icon type="ios-create-outline" size="16" />修改
            </Button>
          </div>
          <div v-else>
            <label>用户名：</label>
            <Input v-model="name" placeholder="请输入用户名..." style="width: 180px" />
            <Button @click="handleOnChange('yhmqx')">取消</Button>
            <Button type="info" ghost @click="handleOnChange('yhmqd')">确定</Button>
          </div>
          <!-- 用户密码 -->
          <div v-if="isPassword">
            <label>用户密码：</label>
            <span>******</span>
            <Button type="dashed" v-if="user.name!=='admin'" @click="handleOnChange('qrmmxg')">
              <Icon type="ios-create-outline" size="16" />修改
            </Button>
          </div>
          <!-- <div v-if="!isPassword">
            <label>用户密码：</label>
            <Input v-model="password" placeholder="请输入旧密码..." style="width: 180px" />
          </div>-->
          <div v-if="!isPassword">
            <label>新密码：</label>
            <Input v-model="newpassword" placeholder="请输入新密码..." style="width: 180px" />
          </div>
          <div v-if="!isPassword">
            <label>确认密码：</label>
            <Input v-model="qrpassword" placeholder="请输入确认密码..." style="width: 180px" />
            <Button @click="handleOnChange('qrmmqx')">取消</Button>
            <Button type="info" ghost @click="handleOnChange('qrmmqd')">确定</Button>
          </div>
          <!-- <div v-if="isEmail">
            <label>邮箱地址：</label>
            <span>{{email}}</span>
            <Button type="dashed" v-if="user.name!=='Admin'" @click="handleOnChange('yxdzxg')">
              <Icon type="ios-create-outline" size="16" />修改
            </Button>
          </div>
          <div v-else >
            <label>邮箱地址：</label>
            <Input v-model="email" placeholder="请输入邮箱地址..." style="width: 180px" />
            <Button @click="handleOnChange('yxdzqx')">取消</Button>
            <Button type="info" ghost @click="handleOnChange('yxdzqd')">确定</Button>
          </div>-->
          <div>
            <label>创建时间：</label>
            <span>{{date}}</span>
          </div>
        </div>
        <div class="tx-box">
          <img :src="urlImg" />
          <Upload
            :action="$url+'/api/person/img'"
            :on-success="handleFileSuccess"
            :show-upload-list="false"
          >
            <Button icon="ios-cloud-upload-outline" v-if="user.name!=='admin'">上传新头像</Button>
          </Upload>
        </div>
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
  data() {
    return {
      user: {}, //模式
      isName: true,
      isPassword: true,
      isEmail: true,

      name: "",
      password: "",
      qrpassword: "",
      newpassword: "",
      email: "",
      url: "",
      urlImg: "",
      bid: "",
      date: ""
    };
  },
  watch: {
    qrpassword(val) {
      this.$Message.destroy();
      if (this.newpassword !== val) {
        this.$Message["error"]({
          background: true,
          content: "新密码与确认密码不一致！"
        });
      }
    }
  },

  mounted() {
    this.setUser();
  },
  methods: {
    setUser() {
      this.user = this.$store.state.variable.info;
      this.name = this.user.name;
      this.bid = this.user.bid;
      this.email = this.user.email;
      this.urlImg = this.user.url;
      //   this.url = this.user.url;
      this.date = this.user.date;
    },
    handleOnChange(key) {
      this.$Message.destroy();
      let data = { bid: this.bid };
      switch (key) {
        case "yhmxg":
          this.isName = false;
          break;
        case "yhmqx":
          this.isName = true;
          break;
        case "yhmqd":
          this.$Message.loading({
            content: "用户名更新中，请稍后...",
            duration: 0
          });
          data.name = this.name;
          this.updateUser(data);
          this.isName = true;
          break;
        case "qrmmxg":
          this.isPassword = false;
          break;
        case "qrmmqx":
          this.isPassword = true;
          break;
        case "qrmmqd":
          this.$Message.loading({
            content: "用户密码更新中，请稍后...",
            duration: 0
          });
          data.password = this.qrpassword;
          this.updateUser(data);
          this.isPassword = true;
          this.password = "";
          this.qrpassword = "";
          this.newpassword = "";
          break;
        case "yxdzxg":
          this.isEmail = false;
          break;
        case "yxdzqx":
          this.isEmail = true;
          break;
        case "yxdzqd":
          this.$Message.loading({
            content: "邮箱地址更新中，请稍后...",
            duration: 0
          });
          data.email = this.email;
          this.updateUser(data);
          this.isEmail = true;
          break;

        default:
          break;
      }
    },
    updateUser(data) {
      this.$axios
        .post("/api/person/user/update", this.$qs.stringify(data))
        .then(res => {
          // this.$Message.destroy();
          if (res.data.result) {
            // this.$Message["success"]({
            //   background: true,
            //   content: "信息更新成功！"
            // });
            this.getUser();
          } else {
            this.$Message["error"]({
              background: true,
              content: "信息更新失败！"
            });
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    getUser() {
      this.$axios
        .get("/api/person/user/list", { params: { bid: this.bid } })
        .then(res => {
          this.$Message.destroy();
          if (res.data.result) {
            this.$Message["success"]({
              background: true,
              content: "信息更新成功！"
            });
            let user = res.data.data[0];
            user.url =
              user.url.indexOf("http") > -1 ? user.url : this.$url + user.url;
            this.$store.commit("setUser", user);
            this.setUser();
          } else {
            this.$Message["error"]({
              background: true,
              content: "信息更新失败！"
            });
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    handleFileSuccess(response, file, fileList) {
      //   data.url = this.email;
      this.updateUser({ bid: this.bid, url: "/" + response.fileInfo.path });
    }
  }
};
</script>
<style lang="scss" scoped>
@import "./index.scss";
</style>
