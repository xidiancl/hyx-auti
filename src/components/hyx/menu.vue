<template>
  <div>
    <v-tabs :active="active" @change="routeto">
      <v-tab header="问诊">
      </v-tab>
      <v-tab header="注射">
      </v-tab>
      <v-tab header="固定表头">
      </v-tab>
      <v-tab header="改变宽度">
      </v-tab>
    </v-tabs>
  </div>
</template>
<script>
  import tab from '../Tabs/Tab'
  import tabs from '../Tabs/Tabset'
  import router from '../../router/index'
  export default{
    data () {
      return {active: 0}
    },
    methods: {
      routeto: function (index) {
        console.log(this)
        if (index === 0) {
          router.push('wenzhen')
        } else if (index === 1) {
          router.push('zhushe')
        } else if (index === 2) {
          router.push('gdbt')
        } else if (index === 3) {
          router.push('gbkd')
        }
      }
    },
    components: {
      'v-tab': tab,
      'v-tabs': tabs
    },
    mounted () {
      console.log(router)
      if (router.currentRoute.name === 'wenzhen') {
        this.active = 0
      } else if (router.currentRoute.name === 'zhushe') {
        this.active = 1
      } else if (router.currentRoute.name === 'gdbt') {
        this.active = 2
      } else if (router.currentRoute.name === 'gbkd') {
        this.active = 3
      }
    }
  }
</script>
