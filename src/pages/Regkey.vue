<template>
  <div class="q-ma-xl">
    <q-markup-table flat bordered>
      <thead class="bg-teal-5">
      <tr>
        <th colspan="5">
          <div class="row no-wrap items-center">
            <div class="text-h4 q-ml-md text-white">注册码列表</div>
          </div>
        </th>
      </tr>
      <tr>
        <th class="text-left">编号</th>
        <th class="text-left">注册码</th>
        <th class="text-right">授权时长</th>
      </tr>
      </thead>
      <tbody class="bg-grey-3" v-for="(item,i) in keyList" :key="i">
      <tr>
        <td class="text-left">{{ i }}</td>
        <td class="text-left">{{ item.key }}</td>
        <td class="text-right">{{ item.length }}</td>
      </tr>
      </tbody>
    </q-markup-table>
    <q-btn label="添加" color="primary" v-on:click="add()"></q-btn>
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
    }
  },
  created () {
    this.updateKey()
  }
}
</script>
