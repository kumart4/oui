<template>
  <div class="oui-layout-header">
    <a-breadcrumb :routes="breadcrumbs">
      <template v-slot:itemRender="{ route }">
        <router-link v-if="route.path" :to="route.path">
          <!-- {{ $t(route.title) }} -->
        </router-link>
        <!-- <span v-else>{{ $t(route.title) }}</span> -->
      </template>
    </a-breadcrumb>
    <div class="header-right">
      <a-tooltip :title="fullscreen ? $t('Exit Full Screen') : $t('Full Screen')">
        <a-icon :type="fullscreen ? 'fullscreen-exit' : 'fullscreen'" style="cursor: pointer; margin-top: 8px; color:#12A1E9; fontSize: 22px;" @click="fullScreen"/>
      </a-tooltip>
      <div>
        <span style="margin-right:10px; fontSize: 16px; color:#12A1E9;">Device ID : <strong>{{deviceID}}</strong></span>
      </div>
      <a-dropdown>
        <a href="javascript:void(0)">
          <span style="margin-right:10px; fontSize: 16px; color:#12A1E9;">{{ $t('Language') }}</span>
          <a-icon type="down"/>
        </a>
        <a-menu slot="overlay" @click="onLangClick" :selectedKeys="selectedLangKeys">
          <a-menu-item key="en">English</a-menu-item>
          <a-menu-item key="zh-cn">简体中文</a-menu-item>
          <a-menu-item key="zh-tw">繁体中文</a-menu-item>
          <a-menu-item key="ja">日本語</a-menu-item>
          <!-- <a-menu-item key="auto">{{ $t('Automatic') }}</a-menu-item> -->
        </a-menu>
      </a-dropdown>
      <!-- <div>
        <a-icon type="info-circle" style="cursor: pointer; margin-top: 8px; color:#12A1E9; fontSize: 22px;" @click="showModalVerInfo"/>
        <a-modal wrap-class-name="version-info-modal" title="Version Info" centered :visible="visibleInfo" :closable="false">
          <template slot="footer">
            <a-button key="submit" type="primary" @click="handleOkInfo">OK</a-button>
          </template>
          <p>{{ versionInfo }}</p>
        </a-modal>
      </div> -->
      <a-dropdown>
        <a href="javascript:void(0)">
          <span style="margin-right:10px; fontSize: 16px; color:#12A1E9;">{{ username }}</span>
          <a-icon type="menu" :style="{ fontSize: '16px' }"/>
        </a>
        <a-menu slot="overlay" @click="onUserClick">
          <a-menu-item key="logout">{{ $t('Logout') }}</a-menu-item>
          <a-menu-item key="reboot">{{ $t('Reboot') }}</a-menu-item>
        </a-menu>
      </a-dropdown>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'

export default {
  data () {
    return {
      breadcrumbs: [],
      username: '',
      deviceID: '',
      versionInfo: '',
      visibleInfo: false
    }
  },
  computed: {
    selectedLangKeys () {
      return [this.lang]
    },
    ...mapState(['lang', 'fullscreen'])
  },
  methods: {
    showModalVerInfo () {
      this.visibleInfo = true
    },
    handleOkInfo () {
      this.visibleInfo = false
    },
    fullScreen () {
      this.$store.commit('toggleFullscreen')
    },
    getBreadCrumbList (route) {
      const homeRoute = this.$router.options.routes.find(r => r.path === '/').children[0]
      // const homeItem = { title: homeRoute.meta.title }
      const homeItem = {}
      if (homeRoute && homeRoute.meta && homeRoute.meta.title) {
        homeItem.title = homeRoute.meta.title
      } else {
        homeItem.title = 'Tantiv4'.toString()
      }
      const matched = route.matched

      if (matched.some(item => item.path === '/home')) {
        return [{
          // title: homeRoute.meta.title,
          // breadcrumbName: homeRoute.meta.title
          title: homeItem.title,
          breadcrumbName: homeItem.title
        }]
      }

      homeItem.path = '/'

      const list = [homeItem, matched[0].meta]

      if (!matched[0].redirect) { list.push(matched[1].meta) }

      return list.map((v, i) => {
        return {
          path: v.path || '',
          breadcrumbName: i + '',
          title: v.title
        }
      })
    },
    onLangClick (item) {
      const lang = item.key
      this.$rpc.call('oui', 'set_lang', { lang: lang }).then(({ lang }) => {
        this.$store.commit('setLang', lang)
        if (lang === 'auto') { lang = navigator.language.toLowerCase() }

        if (lang === 'zh') lang = 'zh-cn'

        if (!this.$i18n.loaded(lang)) {
          this.$rpc.call('oui', 'load_locales', { locale: lang }).then(locales => {
            locales.forEach(locale => this.$i18n.mergeLocaleMessage(lang, locale))
            this.$i18n.locale = lang
          })
        } else {
          this.$i18n.locale = lang
        }
      })
    },
    onUserClick (item) {
      const action = item.key
      if (action === 'logout') {
        this.$router.push('/login')
      } else if (action === 'reboot') {
        this.$confirm({
          title: this.$t('Reboot'),
          content: this.$t('Really reboot now?'),
          onOk: () => {
            this.$system.reboot().then(() => {
              this.$reconnect(this.$t('Rebooting...'))
            })
          }
        })
      }
    }
  },
  watch: {
    '$route' (newRoute) {
      this.breadcrumbs = this.getBreadCrumbList(newRoute)
    }
  },
  created () {
    this.breadcrumbs = this.getBreadCrumbList(this.$route)

    // this.username = this.$session.username()
    // this.username = this.$session.username
    this.username = sessionStorage.getItem('username')

    fetch('device_id.txt')
      .then(response => response.text())
      .then(text => { this.deviceID = text })

    fetch('versionRezrv')
      .then(response => response.text())
      .then(text => { this.versionInfo = text })
  }
}
</script>

<style lang="stylus">
.oui-layout-header {
  margin: 10px 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 24px;

  .ant-breadcrumb-link {
    font-size: 24px;
  }

  .header-right {
    display: flex;

    > * {
      margin-right: 20px;
    }
  }
}

.version-info-modal {
  .ant-modal-header {
    background-color : #05a9e1;
  }
}
</style>
