<template>
  <q-page class="flex flex-center">
    <q-dialog v-model="alert" persistent>
      <q-card class="flex flex-center">

        <div class="q-pa-md" style="width: 480px">

          <q-form
            class="q-gutter-md"
          >
            <q-input
              filled
              v-model="key"
              label="注册码"
              lazy-rules
              :rules="[ val => val && val.length > 0 || '请输入注册码!']"
            />

            <q-input
              filled
              type="number"
              v-model="gid"
              label="群号*"
              lazy-rules
              :rules="[
          val => val !== null && val !== '' || '请输入群号!'
        ]"
            />

            <div>
              <q-btn label="Submit" v-on:click="onSubmit" color="primary"/>
              <q-btn label="Reset" v-on:click="onReset" color="primary" flat class="q-ml-sm" />
            </div>
          </q-form>

        </div>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>

export default {
  name: 'Login',
  data () {
    return {
      key: '',
      gid: 0,
      alert: true
    }
  },
  methods: {
    message (text) {
      this.$q.dialog({
        title: '消息',
        message: text
      }).onOk(() => {
        // console.log('OK')
      }).onCancel(() => {
        // console.log('Cancel')
      }).onDismiss(() => {
        // console.log('I am triggered on both OK and Cancel')
      })
    },
    submit () {
      this.$axios.post('activate?key=' + this.key + '&gid=' + this.gid)
        .then((response) => {
          if (response.data === 'success') {
            this.message('已为您的群' + this.gid.toString() + '注册成功!')
            this.$router.replace('/')
          } else {
            this.message('注册失败!')
          }
        })
        .catch((err) => {
          console.log(err)
        })
    },
    onSubmit () {
      this.$q.dialog({
        title: 'Confirm',
        message: '确定为群' + this.gid.toString() + '注册?',
        cancel: true,
        persistent: true
      }).onOk(() => {
        // console.log('>>>> OK')
      }).onOk(() => {
        this.submit()
      }).onCancel(() => {
        // console.log('>>>> Cancel')
      }).onDismiss(() => {
        // console.log('I am triggered on both OK and Cancel')
      })
    },
    onReset () {
      this.gid = 0
      this.key = ''
    }
  }
}
</script>
