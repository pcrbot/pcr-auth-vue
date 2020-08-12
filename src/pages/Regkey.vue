<template>
  <div class="q-ma-xl">
    <span style="font-size: 28px">注册码列表</span>
    <q-btn label="添加" color="primary" v-on:click="add()"></q-btn>
    <div class="q-pa-md">
      <div class="row justify-center q-gutter-sm" style="width: auto">
        <q-intersection
          v-for="(item,index) in keyList"
          :key="index"
          once
          transition="scale"
          class="example-item"
        >
          <q-card class="bg-primary text-white" style="width: 180px">
            <br>
            <div class="text-bold text-center">
              {{ item.key }}
              <br>
              授权时长:{{ item.length }}天
            </div>
            <br>
            <q-card-section class="justify-center bg-teal-14" style="vertical-align: middle;">
              <q-btn label="Info" v-on:click="alert(index)"></q-btn>
              <q-btn label="Delete" v-on:click="confirm(index)"></q-btn>
            </q-card-section>
          </q-card>
        </q-intersection>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'Regkey',
  data () {
    return {
      keyList: [{
        key: '123',
        length: 23
      }]
    }
  },
  methods: {
    updateKey () {
      this.$axios.get('/get/key')
        .then((response) => {
          this.keyList = response.data
        })
    },
    add () {
      this.$q.dialog({
        title: '添加',
        message: '请填写注册码的授权时长',
        prompt: {
          model: '',
          isValid: val => val.length > 0, // << here is the magic
          type: 'number' // optional
        },
        cancel: true,
        persistent: true
      }).onOk(data => {
        this.$axios.post('addkey?length=' + data)
          .then((response) => {
            if (response) {
              this.updateKey()
            }
          })
          .catch((err) => {
            console.log(err)
          })
      })
    },
    alert (index) {
      this.$q.dialog({
        title: 'INFO',
        message: '您的注册码:' + this.keyList[index].key + '  授权时长:' + this.keyList[index].length + '天'
      }).onOk(() => {
        // console.log('OK')
      }).onCancel(() => {
        // console.log('Cancel')
      }).onDismiss(() => {
        // console.log('I am triggered on both OK and Cancel')
      })
    },
    confirm (index) {
      this.$q.dialog({
        title: 'Confirm',
        message: '此操作将永久删除该注册码, 是否继续?',
        cancel: true,
        persistent: true
      }).onOk(() => {
        // console.log('>>>> OK')
      }).onOk(() => {
        this.$axios.delete('delkey?key=' + this.keyList[index].key)
          .then((response) => {
            if (response.data === 'success') {
              this.updateKey()
            }
          })
      }).onCancel(() => {
        // console.log('>>>> Cancel')
      }).onDismiss(() => {
        // console.log('I am triggered on both OK and Cancel')
      })
    }
  },
  created () {
    this.updateKey()
  }
}
</script>
