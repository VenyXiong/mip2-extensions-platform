<template>
  <mip-fixed
    class="container"
    type="top"
    top="0"
    bottom="0">
    <mip-iframe
      :src="url"
      height="100%"
      width="100%"
      scrolling="yes"
      frameborder="0"
      layout="fill"/>
  </mip-fixed>

</template>

<style scoped>
    .container {
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;
        width: 100%;
        min-height: 505px;
    }
</style>

<script>

import {getQuery, getUUID, getRedirectUrl} from './util'

export default {
  data () {
    /* eslint-disable */
    let redirect_uri = getQuery('redirect_uri')
    let client_id = getQuery('client_id')


    let uuid = getUUID()
    let state = {}
    try {
      state = JSON.parse(getQuery('state'))
    } catch (e) {
      console.error('state的格式不正确')
    }

    return {
      config: {
        redirect_uri,
        client_id,
        state,
        uuid
      },

      url: 'https://openapi.baidu.com/oauth/2.0/mip/authorize?response_type=code' +
                '&scope=snsapi_userinfo' +
                '&confirm_login=2' +
                '&client_id=' + client_id +
                '&redirect_uri=' + encodeURIComponent(redirect_uri) +
                '&state=' + encodeURIComponent(JSON.stringify(state)) +
                '&uuid=' + uuid
    }
    /* eslint-enable */
  },
  created () {
    this.checkConfig()
  },
  mounted () {
    window.addEventListener('message', this.onmessage, false)
  },
  disconnectedCallback () {
    window.removeEventListener('message', this.onmessage, false)
  },
  methods: {
    /**
         * 检查配置
         */
    checkConfig () {
      let config = this.config
      let hasError = false

      if (!config.client_id) {
        this.error('组件必选属性 clientId 为空')
        hasError = true
      } else if (!config.redirect_uri) {
        this.error('组件必选属性 redirect_uri 为空')
        hasError = true
      } else if (!config.state.url) {
        this.error('组件必选属性 state 的内容错误')
        hasError = true
      }

      if (hasError) {
        throw new TypeError('[mip-auth-page] 组件参数检查失败')
      }
    },
    /**
         * 输出错误信息到控制台
         *
         * @param {string} text 输出文本
         */
    error (text) {
      console.error('[mip-auth-page]', text, this)
    },
    onmessage (e) {
      let data = e.data
      let {type, value = {}} = data
      let config = this.config

      if (config.uuid !== value.uuid) {
        return
      }

      if (type === 'oauth-iframe-success') {
        let obj = {
          code: value.verifier,
          state: JSON.stringify(config.state)
        }

        let hash = config.state.h || ''

        window.MIP.viewer.open(
          getRedirectUrl(config.state.url || config.redirect_uri, obj, decodeURIComponent(hash)),
          {isMipLink: true, replace: true}
        )
      }
    }
  }
}
</script>
