<template>
  <div id="app">
    <transition name="fade" mode="out-in">
      <router-view />
    </transition>
  </div>
</template>

<script>
export default {
  name: "app",
  mounted() {
    // console.log(this.$mock);
    //*****************************解决刷新页面数据丢失开始**************************************** */
    if (sessionStorage.getItem("store")) {
      this.$store.replaceState(
        Object.assign(
          {},
          this.$store.state,
          JSON.parse(sessionStorage.getItem("store"))
        )
      );
      sessionStorage.removeItem("store");
    }

    //在页面刷新时将vuex里的信息保存到sessionStorage里
    window.addEventListener("beforeunload", () => {
      this.$socket.close();
      this.$socket.disconnect();
      sessionStorage.setItem("store", JSON.stringify(this.$store.state));
    });
  }
};
</script>
<style lang="scss">
@import "./style/gloable.scss";
</style>

