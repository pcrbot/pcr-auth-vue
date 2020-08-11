<template>
  <q-page class="flex flex-center">
    <q-dialog v-model="alert" persistent>
      <q-card class="flex flex-center">
        <q-card-section>
          <div class="text-h6">请输入管理员密码</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-input v-model="password" type="password"/>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="确定" color="primary" v-on:click="submit"/>
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>

export default {
  name: 'Login',
  data () {
    return {
      password: '',
      alert: true
    }
  },
  methods: {
    submit () {
      this.$axios.post('login?password=' + this.password)
        .then((response) => {
          console.log(response)
          if (response.data === 'success') {
            this.$q.localStorage.set('login', true)
            this.$router.replace('/')
          }
        })
        .catch((err) => {
          console.log(err)
        })
    }
  }
}
</script>
