<template>
  <q-page class="doc-page">
    <div class="q-pa-md">
      <q-table
        title="Treats"
        :data="data"
        :columns="columns"
        row-key="gid"
        selection="multiple"
        :selected.sync="selected"
        :filter="filter"
      >
        <template v-slot:top>
          <div class="q-table__title row">
            授权群组管理
          </div>
          <q-space/>
          <div class="row">
            <q-input borderless dense debounce="300" v-model="filter" placeholder="Search">
              <template v-slot:append>
                <q-icon name="search" />
              </template>
            </q-input>
            <q-btn class="q-mr-xs" color="info" v-on:click="updateGroup">刷新</q-btn>
            <q-btn class="q-mr-xs" color="secondary" @click="modifySelect()">修改</q-btn>
            <q-btn class="q-mr-xs" color="primary" @click="isAdd = true">添加授权</q-btn>
            <q-btn class="q-mr-xs" color="accent" @click="broadcastSelect">提醒续费</q-btn>
            <q-btn class="q-mr-xs" color="negative" @click="gunSelect">退群</q-btn>
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
              v-model="addGid"
              label="群号"
              lazy-rules
              :rules= "[val => val !== null && val > 0, '' || '请正确的群号!']"
            />
            <q-input
              filled
              type="number"
              v-model="duration"
              label="时长"
              lazy-rules
              :rules= "[val => val !== null && val >0, '' || '请正确的时长!']"
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
          name: 'gid',
          required: true,
          label: '群号',
          align: 'left',
          field: row => row.gid,
          format: val => `${val}`,
          sortable: true,
          sort: (a, b) => parseInt(a, 20) - parseInt(b, 20)
        },
        {
          name: 'groupName',
          required: true,
          label: '群名称',
          align: 'left',
          field: row => row.groupName,
          format: val => `${val}`
        },
        { name: 'deadline', align: 'right', label: '截至日期', field: 'deadline', sortable: true }
      ],
      data: [
      ],
      isAdd: false,
      addGid: 123456,
      duration: 30
    }
  },
  methods: {
    updateGroup () {
      this.$axios.get('/get/group?password=' + LocalStorage.getItem('password'))
        .then((response) => {
          this.data = response.data
          console.log(response.data)
        })
    },
    modifySelect () {
      this.$q.dialog({
        title: '修改',
        message: '请填写群组增加(减少)授权时长',
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
        this.updateGroup()
        this.message('成功', '修改完毕!')
      })
    },
    modifyItem (item, duration) {
      this.$axios.post('update/group?gid=' + item.gid + '&duration=' + duration)
    },
    message (title, text) {
      this.$q.dialog({
        title: title,
        message: text,
        html: true
      })
    },
    onSubmit () {
      this.$axios.post('add/group?gid=' + this.addGid + '&duration=' + this.duration)
        .then((response) => {
          if (response) {
            this.message('成功', '添加完成!')
            this.isAdd = false
            this.updateGroup()
          }
        })
        .catch((err) => {
          console.log(err)
        })
    },
    broadcastSelect () {
      this.$q.dialog({
        title: '请填写推送语',
        prompt: {
          model: '您的机器人即将到期，请及时续费!',
          type: 'text' // optional
        },
        cancel: true
      }).onOk(data => {
        this.selected.forEach((item) => {
          this.broadcastItem(item, data)
        })
      })
    },
    broadcastItem (item, msg) {
      this.$axios.post('/notify/group?gid=' + item.gid + '&msg=' + msg)
        .then((response) => {
          if (response.data === 'failed') {
            this.$q.notify({
              message: '群' + item.gid + '推送失败!',
              position: 'top',
              color: 'purple'
            })
          } else {
            this.$q.notify({
              message: '群' + item.gid + '推送成功!',
              position: 'top',
              color: 'green'
            })
          }
        }).catch(() => {
          this.$q.notify({
            message: '群' + item.gid + '推送失败!',
            position: 'top',
            color: 'purple'
          })
        })
    },
    gunSelect () {
      this.$q.dialog({
        title: '确认',
        message: '确定退出群聊吗?',
        cancel: true
      }).onOk(data => {
        this.selected.forEach((item) => {
          this.gunItem(item)
        })
        this.updateGroup()
      })
    },
    gunItem (item) {
      this.$axios.post('/gun/group?gid=' + item.gid)
        .then((response) => {
          if (response.data === 'failed') {
            this.$q.notify({
              message: '群' + item.gid + '退出失败!',
              position: 'top',
              color: 'purple'
            })
          } else {
            this.$q.notify({
              message: '群' + item.gid + '退出成功!',
              position: 'top',
              color: 'green'
            })
          }
        }).catch(() => {
          this.$q.notify({
            message: '群' + item.gid + '推送失败!',
            position: 'top',
            color: 'purple'
          })
        })
    }
  },
  created () {
    this.updateGroup()
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
