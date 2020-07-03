<template>
  <div id="app">
    <mo-title-bar
      v-if="isRenderer"
      :showActions="showWindowActions"
    />
    <router-view />
    <mo-engine-client
      :secret="rpcSecret"
    />
    <mo-ipc v-if="isRenderer" />
    <mo-dynamic-tray v-if="enableTraySpeedometer" />
  </div>
</template>

<script>
  import is from 'electron-is'
  import { mapState } from 'vuex'
  import { getLangDirection } from '@shared/utils'
  import { APP_THEME } from '@shared/constants'
  import DynamicTray from '@/components/Native/DynamicTray'
  import EngineClient from '@/components/Native/EngineClient'
  import Ipc from '@/components/Native/Ipc'
  import TitleBar from '@/components/Native/TitleBar'

  export default {
    name: 'Motrix',
    components: {
      [DynamicTray.name]: DynamicTray,
      [EngineClient.name]: EngineClient,
      [Ipc.name]: Ipc,
      [TitleBar.name]: TitleBar
    },
    computed: {
      isMac: () => is.macOS(),
      isRenderer: () => is.renderer(),
      ...mapState('app', {
        systemTheme: state => state.systemTheme
      }),
      ...mapState('preference', {
        showWindowActions: state => {
          return (is.windows() || is.linux()) && state.config.hideAppMenu
        },
        traySpeedometer: state => state.config.traySpeedometer,
        rpcSecret: state => state.config.rpcSecret,
        theme: state => state.config.theme,
        locale: state => state.config.locale,
        dir: state => getLangDirection(state.config.locale)
      }),
      themeClass () {
        if (this.theme === APP_THEME.AUTO) {
          return `theme-${this.systemTheme}`
        } else {
          return `theme-${this.theme}`
        }
      },
      i18nClass () {
        return `i18n-${this.locale}`
      },
      dirClass () {
        return `dir-${this.dir}`
      },
      rootClass () {
        const { themeClass = '', i18nClass = '', dirClass = '' } = this
        const result = `${themeClass} ${i18nClass} ${dirClass}`
        return result
      },
      enableTraySpeedometer () {
        const { traySpeedometer, isMac, isRenderer } = this
        return traySpeedometer && isMac && isRenderer
      }
    },
    methods: {
      updateRootClassName () {
        const { rootClass } = this
        document.documentElement.className = rootClass
      }
    },
    beforeMount () {
      this.updateRootClassName()
    },
    watch: {
      rootClass (val, oldVal) {
        this.updateRootClassName()
      }
    }
  }
</script>

<style>
</style>
