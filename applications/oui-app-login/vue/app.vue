<template>

  <div>
    <div class="header">
      <div class="center">
        <img src="/icons/REZRV-Logo.png"/>
        <p style="font-size: 30px;">ROUTER LOGIN</p>
      </div>
    </div>

  <div>
    <a-spin size="large" class="oui-login" :spinning="spinning">
      <a-card :title="$t('login.Authorization Required')">
        <a-form-model :model="form">
          <a-form-model-item prop="username">
            <a-input v-model="form.username" @pressEnter="handleLogin" :placeholder="$t('login.Please input username')">
              <a-icon slot="prefix" type="user"/>
            </a-input>
          </a-form-model-item>
          <a-form-model-item prop="password">
            <a-input-password v-model="form.password" @pressEnter="handleLogin" type="password" :placeholder="$t('login.Please input password')">
              <a-icon slot="prefix" type="lock"/>
            </a-input-password>
          </a-form-model-item>
          <a-form-model-item>
            <a-button type="primary" size="large" block @click="handleLogin">{{ $t("login.Login") }}</a-button>
          </a-form-model-item>
        </a-form-model>
      </a-card>
    </a-spin>
      <div style="position: fixed;  top: 69%; left: 39%; font-weight: bold; font-size: 15px;">
        Download REZRV Apps from the links below:
      </div>
    </div>

    <div style = " width: 100%;
    top: 74%;
    left: 35%;
    height: 13%;
    position: fixed;">
      <img src="/icons/rezrvAndroidAppQRCode.png" width="7%"/>
      <div style="position: fixed; left: 34%; font-weight: bold; font-size: 15px;">Rezrv Android App</div>
    </div>

    <div style = " width: 100%;
    top: 74%;
    left: 58%;
    height: 13%;
    position: fixed;">
      <img src="/icons/rezrvIOSAppQRCode.png" width="7%"/>
      <div style="position: fixed; left: 58%; font-weight: bold; font-size: 15px;">Rezrv iOS App</div>
    </div>

    <div style = " width: 100%;
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
    return {
      form: {
        username: '',
        password: ''
      },
      spinning: false
    }
  },
  methods: {
    handleLogin () {
      this.spinning = true
      this.$session.login(this.form.username, this.form.password).then((ok) => {
        if (ok) {
          this.$router.push('/')
          return
        }
        this.$message.error(this.$t('login.Wrong password given!'), 1)
        this.spinning = false
      })
    }
  },
  created () {
    this.$session.logout()
  }
}
</script>

<style>

.header {
  top: 2%;
  left: 30%;
  position: fixed;
  width: 41%;
}

.center {
  text-align: center;
}

.oui-login {
  width: 500px;
  top: 55%;
  left: 50%;
  position: fixed;
  transform: translate(-50%, -50%);
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
