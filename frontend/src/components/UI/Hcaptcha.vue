<template>
  <div>
    <div id='hcap-script' />
  </div>
</template>
<script>
// https://github.com/hCaptcha/vue-hcaptcha/blob/master/src/component/vue-hcaptcha.vue

const CaptchaScript = (cb) => {
  let script = document.createElement('script')
  script.src = 'https://hcaptcha.com/1/api.js?render=explicit'
  script.async = true

  script.addEventListener('load', cb, true)
  return script
}
module.exports = {
  name: 'VueHcaptcha',
  props: {
    size: {
      type: String
    },
    tabindex: {
      type: String
    }
  },
  mounted () {
    if (typeof window.hcaptcha === 'undefined') { // if not loaded, create script tag, and wait to render hcaptcha element
      let script = CaptchaScript(this.onloadScript)
      let container = document.getElementById('hcap-script')
      if (document.getElementsByTagName('head').length > 0) {
        container = document.getElementsByTagName('head')[0]
      }
      container.appendChild(script) // append this here, this appends the tag to the start of the app.
    }
    else {
      this.onloadScript()
    }
  },
  methods: {
    onloadScript () {
      // set options for VueHcaptcha to be passed to the onload script
      let opt = {
        sitekey: '69d55f48-8b59-4e20-9dc5-a865baac19cf',
        theme: this.$store.state.theme === 'dark' ? 'dark' : 'light',
        size: this.size ? this.size : '',
        tabindex: this.tabindex ? this.tabindex : '',
        callback: this.onVerify,
        'expired-callback': this.onExpired,
        'error-callback': this.onError
      }
      // Render hCaptcha widget and provide neccessary callbacks
      if (typeof window.hcaptcha !== 'undefined') {
        let hcaptcha = window.hcaptcha // convienence var to access
        let container = this.$slots.default ? this.$el.children[0] : this.$el
        this.$widgetId = hcaptcha.render(container, opt)
      }
    },
    onError (e) {
      if (window.hcaptcha === 'undefined') {
        return
      }
      this.$emit('error', e)
      this.reset() // reset the captcha
    },
    // let user handle the errors, etc
    onVerify (response) {
      this.$emit('verify', response)
    },
    onExpired () {
      this.$emit('expired')
    },
    execute () {
      window.hcaptcha.execute(this.$widgetId)
    },
    reset () {
      window.hcaptcha.reset(this.$widgetId)
    }
  }
}
</script>
