<template>
  <div class="q-ma-xl">
    <span style="font-size: 28px">授权列表</span>
    <div class="q-pa-md">
      <div class="row justify-center q-gutter-sm" style="width: auto">
        <q-intersection
          v-for="(item,index) in groupList"
          :key="index"
          once
          transition="scale"
          class="example-item"
        >
          <q-card class="bg-primary text-white" style="width: 180px">
            <br>
            <div class="text-bold text-center">
              群号: {{ item.gid }}
            </div>
            <br>
            <q-card-section class="justify-center bg-teal-14" style="vertical-align: middle;">
              授权截止至: {{ item.deadline }}
            </q-card-section>
          </q-card>
        </q-intersection>
      </div>
    </div>
  </div>
</template>

<script>
import { LocalStorage } from 'quasar'

export default {
  name: 'Group',
  data () {
    return {
      groupList: [{
        gid: 123456,
        deadline: '2020-12-20'
      }]
    }
  },
  methods: {
    updateGroup () {
      this.$axios.get('/get/group?password=' + LocalStorage.getItem('password'))
        .then((response) => {
          this.groupList = response.data
        })
    }
  },
  created () {
    this.updateGroup()
  }
}
</script>

<style scoped>

</style>
