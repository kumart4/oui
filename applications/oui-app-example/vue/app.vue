<template>
  <!-- <div class="example">
    <a-input-search :placeholder="$t('example.Please input text')" :enter-button="$t('example.Send')" size="large" @search="send"/>
    <a-input v-model="resp"/>
  </div> -->
  <div>
    <iframe class="iframe-tantiv" src="http://10.3.12.1:8089/"></iframe>
  </div>
</template>

<script>
export default {
  data () {
    return {
      resp: ''
    }
  },
  methods: {
    send (value) {
      this.$rpc.call('example', 'echo', { text: value }).then(r => {
        this.resp = r.text
      })
    }
  }
}
</script>

<style>
.example {
  width: 500px;
}

.example .ant-input-search {
  margin-bottom: 20px;
}

.iframe-tantiv {
  height: 100vh;
  width: 100%;
  border: none;
}
</style>
