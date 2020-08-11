<template>
  <div id="q-app">
    <router-view />
  </div>
</template>
<script>
export default {
  name: 'App',
  created () {
    if (this.$q.localStorage.getItem('login') !== true) {
      this.$router.push('/login')
    }
  }
}
</script>
