<template>
  <q-page class="doc-page">
    <div class="q-pa-md">
      <q-table
        title="Treats"
        :data="data"
        :columns="columns"
        row-key="key"
        selection="multiple"
        :selected.sync="selected"
        :filter="filter"
      >
        <template v-slot:top>
          <div class="q-table__title row">
            充值卡密管理
          </div>
          <q-space/>
          <div class="row">
            <q-input borderless dense debounce="300" v-model="filter" placeholder="Search">
              <template v-slot:append>
                <q-icon name="search" />
              </template>
            </q-input>
            <q-btn class="q-mr-xs" color="positive" v-on:click="output">导出</q-btn>
            <q-btn class="q-mr-xs" color="info" v-on:click="updateKey">刷新</q-btn>
            <q-btn class="q-mr-xs" color="secondary" @click="isAdd = true">添加</q-btn>
            <q-btn class="q-mr-xs" color="primary" @click="modifySelect()">修改</q-btn>
            <q-btn class="q-mr-xs" color="negative" @click="deleteSelect">删除</q-btn>
          </div>
        </template>
      </q-table>
    </div>
    <q-dialog v-model="isAdd">
      <q-card class="flex flex-center">
        <div class="q-pa-md" style="width: 480px">
          <q-form
            class="q-gutter-md"
          >
            <q-input
              filled
              type="number"
              v-model="duration"
              label="时长"
              lazy-rules
              :rules= "[val => val !== null && val > 0, '' || '请正确的时长!']"
            />
            <q-input
              filled
              type="number"
              v-model="num"
              label="数量"
              lazy-rules
              :rules= "[val => val !== null && val >0, '' || '请正确的数量!']"
            />

            <div align="center">
              <q-btn label="Submit" @click="onSubmit()" color="primary"/>
            </div>
          </q-form>
        </div>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
import { LocalStorage } from 'quasar'

export default {
  data () {
    return {
      filter: '',
      selected: [],
      columns: [
        {
          name: 'key',
          required: true,
          label: '充值卡密',
          align: 'left',
          field: row => row.key,
          format: val => `${val}`,
          sortable: true
        },
        { name: 'duration', align: 'right', label: '时长', field: 'duration', sortable: true, sort: (a, b) => parseInt(a, 10) - parseInt(b, 10) }
      ],
      data: [
      ],
      isAdd: false,
      num: 1,
      duration: 30
    }
  },
  methods: {
    updateKey () {
      this.$axios.get('/get/key?password=' + LocalStorage.getItem('password'))
        .then((response) => {
          this.data = response.data
        })
    },
    deleteSelect () {
      this.$q.dialog({
        title: 'Confirm',
        message: '此操作将永久删除, 是否继续?',
        cancel: true,
        persistent: true
      }).onOk(() => {
        // console.log('>>>> OK')
      }).onOk(() => {
        this.selected.forEach((item) => {
          this.deleteItem(item)
        })
        this.updateKey()
        this.message('成功', '删除完毕!')
      })
    },
    deleteItem (item) {
      this.$axios.delete('del/key?key=' + item.key)
    },
    modifySelect () {
      this.$q.dialog({
        title: '修改',
        message: '请填写注册码的授权时长',
        prompt: {
          model: '',
          isValid: val => val.length > 0, // << here is the magic
          type: 'number' // optional
        },
        cancel: true,
        persistent: true
      }).onOk(data => {
        this.selected.forEach((item) => {
          this.modifyItem(item, data)
        })
        this.updateKey()
        this.message('成功', '修改完毕!')
      })
    },
    modifyItem (item, duration) {
      this.$axios.post('update/key?key=' + item.key + '&duration=' + duration)
    },
    message (title, text) {
      this.$q.dialog({
        title: title,
        message: text,
        html: true
      })
    },
    output () {
      let msg = ''
      this.selected.forEach((item) => {
        msg = msg + '卡密:' + item.key + '<br>时长:' + item.duration + '天<br>'
      })
      this.message('导出卡密', msg)
    },
    onSubmit () {
      this.$axios.post('add/key?duration=' + this.duration + '&num=' + this.num)
        .then((response) => {
          if (response) {
            this.message('成功', '添加完成!')
            this.isAdd = false
            this.updateKey()
          }
        })
        .catch((err) => {
          console.log(err)
        })
    }
  },
  created () {
    this.updateKey()
  }
}
</script>

<style>
.doc-page {
  padding: 16px 46px;
  font-weight: 300;
  max-width: 900px;
  margin-left: auto;
  margin-right: auto;
}
</style>
