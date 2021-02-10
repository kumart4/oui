<template>
  <div style="padding: 20px 0 0 0;">
    <a-steps style="padding: 20px ;" :current="current">
      <a-step v-for="(s, i) in steps" :key="i" :title="$t(s)"/>
    </a-steps>
    <div class="header">
      <div class="center">
        <img src="/icons/REZRV-Logo.png"/>
      </div>
    </div>
    <div class="steps-content">
      <a-card :title="$t(steps[current])">
        <a-form-model style="width: 400px" ref="form" :model="form" :rules="rules">
          <a-form-model-item prop="lang" v-if="current === 0">
            <a-select v-model="form.lang" @change="langChanged">
              <a-select-option value="en">English</a-select-option>
              <a-select-option value="zh-cn">简体中文</a-select-option>
              <a-select-option value="zh-tw">繁体中文</a-select-option>
              <a-select-option value="ja">日本語</a-select-option>
              <!-- <a-select-option value="auto">{{ $t('Automatic') }}</a-select-option> -->
            </a-select>
          </a-form-model-item>
          <div v-if="current === 1">
            <a-form-model-item :label="$t('wizard.Username')">
              <a-input value="admin" readonly/>
            </a-form-model-item>
            <a-form-model-item :label="$t('wizard.Password')" prop="password">
              <a-input-password v-model="form.password"/>
            </a-form-model-item>
            <a-form-model-item :label="$t('wizard.Confirmation')" prop="confirm">
              <a-input-password v-model="form.confirm"/>
            </a-form-model-item>
          </div>

          <div v-if="current === 2">
            <a-form-model :model="formChk" :label-col="labelCol" :wrapper-col="wrapperCol">
              <a-form-model-item :label="$t('wizard.Do you want to configure SSID and Password')">
                <a-switch v-model="formChk.delivery" />
              </a-form-model-item>
              <div v-if="formChk.delivery === true">
                <div style="float: center; padding: 0 0 0 20px;">
                  <p style="font-size: 120%; ">{{ $t('wizard.Save Apply the Changes') }}</p>
                  <!-- <a-form-model-item label="check for both 2.4Ghz and 5GHz SSID same ">
                    <a-checkbox @change="onChangeCheckBox">
                    </a-checkbox>
                  </a-form-model-item> -->
                </div>
                <oui-form uci-config="wireless">
                  <div v-if="radios.length > 0" name="top" :animated="false">
                    <div style="position: relative; padding-left:40px;" v-for="radio in radios" :key="radio.name" :tab="radio.name + ` (${radio.hardware})`">
                      <oui-typed-section style="border: none;" type="wifi-iface" :filter="self => filterCheck(self, radio)" v-slot="{ s }"
                                      :teasers="['ssid']" :collapsible="false">
                        <div style="font-size: 120%;"><strong>{{radio.channel > 13? 'SSID 5Ghz' : 'SSID 2.4Ghz'}}</strong></div>
                        <br/>
                        <div v-if="formChk.checkboxVal === true">
                          <!-- <p>Check val true</p> -->
                        </div>
                        <div v-if="formChk.checkboxVal === false">
                          <oui-form-item-input :uci-section="s" label="SSID" rules="ssidValidator" name="ssid" required/>
                          <div v-if="s.encryption !== 'none'">
                            <oui-form-item-input :uci-section="s" label="Password" name="key" rules="passwordValidator" password/>
                          </div>
                          <hr>
                        </div>
                        <!-- <oui-form-item-input :uci-section="s" label="SSID" name="ssid" />
                        <div v-if="s.encryption !== 'none'">
                          <oui-form-item-input :uci-section="s" label="Password" name="key" password/>
                        </div>
                        <hr> -->
                      </oui-typed-section>
                    </div>
                  </div>
                </oui-form>
              </div>
            </a-form-model>
          </div>

        </a-form-model>
      </a-card>
      <div class="steps-action">
        <a-button type="primary" :disabled="current === 0" @click="prev">{{ $t('Back') }}</a-button>
        <a-button type="primary" v-if="current === 0 || ( current < steps.length - 1 && form.password !== '' && form.password.length <= 64 && form.password === form.confirm )" @click="next">{{ $t('Next') }}</a-button>
        <a-button type="primary" v-if="current === steps.length - 1" @click="submit">{{ $t('wizard.Submit') }}</a-button>
      </div>
    </div>
    <div style = "width: 100%;
      top: 89%;
      height: 11%;
      position: fixed;
      background-color: #05a9e1; ">
      <div class="footer">
        <a target="_blank" style="color: #ffffff;">Powered By</a>
      </div>

      <div class="footer_1">
        <img src="/icons/Tantiv-Logo.png" width="170" height="40" />
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data () {
    const validatePass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error(this.$t('wizard.Please enter your password')))
      } else if (value.length > 64) {
        callback(new Error(this.$t('wizard.password length must be less then 64 characters')))
      } else {
        if (this.form.confirm !== '') { this.$refs.form.validateField('confirm') }
        callback()
      }
    }

    const validatorConfirm = (rule, value, callback) => {
      if (value === '') {
        callback(new Error(this.$t('wizard.Please enter your password again')))
      } else if (value.length > 64) {
        callback(new Error(this.$t('wizard.password length must be less then 64 characters')))
      } else if (value !== this.form.password) {
        callback(new Error(this.$t('wizard.Inconsistent input password twice!')))
      } else {
        callback()
      }
    }

    return {
      current: 0,
      steps: [
        'Select Your Language',
        'Set administrator password',
        'Router SSID and Security Configuration'
      ],
      form: {
        lang: 'en',
        password: '',
        confirm: ''
      },
      rules: {
        password: [{ validator: validatePass }],
        confirm: [{ validator: validatorConfirm }]
      },
      radios: [],
      new_s_val: [],
      modes: [
        ['ap', this.$t('Access Point')],
        ['sta', this.$t('Client')],
        ['adhoc', this.$t('Ad-Hoc')]
      ],
      interfaces: [],
      encryptions: [
        ['none', this.$t('No encryption')],
        ['psk', 'WPA-PSK'],
        ['psk2', 'WPA2-PSK'],
        ['psk-mixed', 'WPA/WPA2-PSK ' + this.$t('mixed')]
      ],
      ciphers: [
        ['auto', this.$t('auto')],
        ['ccmp', this.$t('Force CCMP (AES)')],
        ['tkip', this.$t('Force TKIP')],
        ['tkip+ccmp', this.$t('Force TKIP and CCMP (AES)')]
      ],
      macfilters: [
        ['allow', this.$t('Allow listed only')],
        ['deny', this.$t('Allow all except listed')]
      ],
      labelCol: { span: 20 },
      wrapperCol: { span: 1 },
      formChk: {
        delivery: false,
        checkboxVal: false,
        empty: ''
      },
      accessSSIDList: false
    }
  },
  created () {
    var lang = 'en'
    this.$store.commit('setLang', lang)

    if (!this.$i18n.loaded(lang)) {
      this.$rpc.call('oui', 'load_locales', { locale: lang }).then(locales => {
        locales.forEach(locale => this.$i18n.mergeLocaleMessage(lang, locale))
        this.$i18n.locale = lang
      })
    } else {
      this.$i18n.locale = lang
    }
  },
  methods: {
    filterCheck (s, radio) {
      if (s.device === radio.name && (s.disabled === undefined || s.disabled === '0')) {
        console.log('filterCheck = ', s, radio)
        const obj = { ssid: s.ssid, key: s.key }
        this.new_s_val.push(obj)
        return s
      }
    },
    onChangeCheckBox (e) {
      console.log('ChangeCheckBox = ', e.target.checked)
      this.formChk.checkboxVal = e.target.checked
      console.log('new_s_val = ', this.new_s_val)
    },
    next () {
      console.log('this.current = ', this.current)

      if (this.current === 1 && this.accessSSIDList === false) {
        this.accessSSIDList = true
        this.$refs.form.validate(valid => {
          if (!valid) return
          this.$rpc.call('oui', 'first_set', {
            lang: this.form.lang,
            username: 'admin',
            password: this.form.password
          }).then(() => {
            sessionStorage.setItem('__oui_first_login', 'false')
            this.current++

            this.$session.login('admin', this.form.password).then((ok) => {
              if (ok) {
                this.$uci.load('wireless').then(() => {
                  const sections = this.$uci.sections('wireless', 'wifi-device')
                  let radiosNum = sections.length

                  console.log('radiosNum = ', radiosNum)
                  sections.forEach(s => {
                    const device = s['.name']
                    const promises = []

                    promises.push(this.$rpc.ubus('iwinfo', 'info', { device }))
                    promises.push(this.$rpc.ubus('iwinfo', 'freqlist', { device }))
                    promises.push(this.$rpc.ubus('iwinfo', 'txpowerlist', { device }))
                    promises.push(this.$rpc.ubus('iwinfo', 'countrylist', { device }))

                    Promise.all(promises).then(rs => {
                      // console.log('rs = ', rs)
                      const channels = [['auto', this.$t('Automatic')]]
                      const info = rs[0]
                      const freqlist = rs[1].results
                      const txpowerlist = []
                      const countrylist = []

                      if (freqlist) {
                        freqlist.forEach(f => {
                          if (f.restricted) { return }
                          channels.push([f.channel, `${f.channel} (${f.mhz} MHz)`])
                        })
                      }

                      if (rs[2] && rs[2].results) {
                        rs[2].results.forEach(tx => {
                          txpowerlist.push([tx.dbm, `${tx.dbm} dBm (${tx.mw} mW)`])
                        })
                      }

                      if (rs[3] && rs[3].results) {
                        rs[3].results.forEach(c => {
                          countrylist.push([c.code, `${c.code} - ${c.country}`])
                        })
                      }

                      const hwmodes = ['11g']

                      if (info && info.hwmodes) {
                        if (info.hwmodes.indexOf('a') > -1 || info.hwmodes.indexOf('ac') > -1) { hwmodes.push('11a') }

                        var objRadioPush = {
                          name: device,
                          channel: info.channel,
                          txpower: info.txpower,
                          country: info.country,
                          hardware: info.hardware.name,
                          hwmodes: hwmodes,
                          htmodes: info.htmodes,
                          channels: channels,
                          txpowerlist: txpowerlist,
                          countrylist: countrylist,
                          ssidName: ''
                        }

                        console.log('s = ', s)
                        console.log('objRadioPush = ', objRadioPush)
                        this.radios.push(objRadioPush)
                      }

                      radiosNum--

                      if (radiosNum === 0) {
                        // this.$spin(false)
                      }
                    })
                  })
                })

                this.$network.load().then(() => {
                  const interfaces = this.$network.getInterfaces()
                  console.log('interfaces = ', interfaces)
                  this.interfaces = interfaces.map(item => item.name)
                })
              }
            })
          })
        })
      } else {
        this.current++
      }
    },
    prev () {
      this.current--
    },
    langChanged (lang) {
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
    },
    submit () {
      this.$router.push('/')
      // this.$router.push('/login')
      // this.$refs.form.validate(valid => {
      //   if (!valid) return
      //   this.$rpc.call('oui', 'first_set', {
      //     lang: this.form.lang,
      //     username: 'admin',
      //     password: this.form.password
      //   }).then(() => {
      //     sessionStorage.setItem('__oui_first_login', 'false')
      //     this.$router.push('/login')
      //   })
      // })
    },
    AddIface (self, radio) {
      const sid = self.addSection()
      self.set(sid, 'device', radio)
    },
    loadEncr (self) {
      const [v] = (this.$uci.get('wireless', self.sid, 'encryption') || '').split('+')
      return v
    },
    loadCipher (self) {
      let v = (this.$uci.get('network', self.sid, 'encryption') || '').split('+')

      if (v.length < 2) { return 'auto' }

      v = v.slice(1).join('+')

      if (v === 'aes') {
        v = 'ccmp'
      } else if (v === 'tkip+aes' || v === 'aes+tkip' || v === 'ccmp+tkip') {
        v = 'tkip+ccmp'
      }

      return v
    },
    saveEncr (self) {
      const cipher = this.$uci.get('network', self.sid, 'cipher')
      let value = self.model

      if (cipher === 'tkip' || cipher === 'ccmp' || cipher === 'tkip+ccmp') {
        value = `${value}+${cipher}`
      }

      this.$uci.set('network', self.sid, 'encryption', value)
    },
    saveCipher (self) {
      let encr = this.$uci.get(self.sid, 'encryption')
      const value = self.model

      if (value === 'tkip' || value === 'ccmp' || value === 'tkip+ccmp') {
        encr = `${encr}+${value}`
      }

      this.$uci.set('network', self.sid, 'encryption', encr)
    }
  }
}
</script>
<style scoped lang="stylus">
.steps-content {
  padding-top: 50px;
  width: 500px;
  height: 75%;
  left: 50%;
  top: 13%;
  position: fixed;
  transform: translate(-50%, 0);
  overflow-y:scroll;
  -ms-overflow-style: none;  /* IE and Edge */
  scrollbar-width: none;  /* Firefox */
}

.steps-content::-webkit-scrollbar {
  width: 0px;  /* Remove scrollbar space */
  background: transparent;  /* Optional: just make scrollbar invisible */
}

.steps-action {
  margin-top: 10px;
  text-align: center;
  > * {
    margin-right: 20px;
  }
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

.header {
  top: 30%;
  left: 1%;
  height: 25%;
  position: fixed;
  width: 25%;
}

.center {
  text-align: center;
}
</style>
