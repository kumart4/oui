<template>
  <a-layout>
    <a-layout-sider :style="{ overflow: 'auto', height: '100vh'}">
      <oui-side/>
    </a-layout-sider>
    <a-layout>
      <a-layout-content style="padding: 0 16px 16px; height: 100vh">
        <oui-header/>
        <div ref="oui-main-content" class="oui-main-content">
          <transition name="main" mode="out-in">
            <router-view></router-view>
          </transition>
          <a-back-top :target="() => $refs['oui-main-content']"/>
        </div>
        <!-- <div style="position: fixed; bottom: 10px; right: 40px">
          <a href="https://github.com/zhaojh329/oui" target="_blank">Powered by Tantiv4</a>
        </div> -->
      </a-layout-content>
      <div style = " width: 100%;
        top: 90%;
        height: 10%;
        position: fixed;
        background-color: #05a9e1; ">
        <div class="footer">
          <a target="_blank" style="color: #ffffff;">Powered By</a>
        </div>

        <div class="footer_1">
          <img src="/icons/Tantiv-Logo.png" width="170" height="40" />
        </div>
        </div>
    </a-layout>
  </a-layout>
</template>
<script>
import OuiSide from './OuiSide.vue'
import OuiHeader from './OuiHeader'

export default {
  components: {
    OuiSide,
    OuiHeader
  },
  computed: {
    hostname () {
      return this.$store.state.hostname
    }
  },
  watch: {
    hostname () {
      document.title = this.hostname
    }
  },
  created () {
    this.$system.getBoardInfo().then(r => {
      this.$store.commit('setHostname', r.hostname)
    })
  }
}
</script>

<style scoped>
.main-enter, .main-leave-to {
  opacity: 0;
  transform: translateY(30px);
}

.main-enter-active {
  transition: all 0.2s;
}

.main-leave-active {
  position: absolute;
  transition: all 0.3s;
}

.oui-main-content {
  overflow: hidden;
  overflow-y: visible;
  padding: 15px;
  background-color: white;
  /* border: 1px solid red; */
  height: calc(100vh - 150px);
  -ms-overflow-style: none;  /* IE and Edge */
  scrollbar-width: none;  /* Firefox */
}

.oui-main-content::-webkit-scrollbar {
  width: 0px;  /* Remove scrollbar space */
  background: transparent;  /* Optional: just make scrollbar invisible */
}

.footer {
  font-size: 25px;
  position: fixed;
  bottom: 50px;
  right: 20px;
  color: #ffffff;
}

.footer_1 {
  font-size: 25px;
  position: fixed;
  bottom: 10px;
  right: 2px;
  color: white;
}
</style>
