<template>
  <div id="app">
    <img src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>

    <hosts-panel
      v-if="hostsPanelShow"
      :title="transferTitle"
      :origin-target="originTarget"
      :symbol="hostSymbol"
      @save="saveHostsPanel"
      @cancel="cancelHostsPanel"
      @closeDialog="closeHostsPanel"
    ></hosts-panel>

  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import hostsPanel from './components/hostsPanel.vue'

export default {
  name: 'app',
  data: function() {
    return {
      hostsPanelShow: false,
      transferTitle: 'hostsPanel组件标题',
      originTarget: [],
      hostSymbol: 'symbol'
    }
  },
  method: {
    saveHostsPanel: function(symbol, targetLists) {
        /* 保存数据 */
        console.log('saveHostsPanel -', symbol, targetLists)
        this.closeHostsPanel()
    },
    cancelHostsPanel: function() {
        /* 取消 */
        this.closeHostsPanel()
    },
    closeHostsPanel: function() {
        /* 取消 */
        this.hostsPanelShow = false
    },
    showHostPanel() {
      this.hostsPanelShow = true
      this.transferTitle = 'HostsPanel组件标题'
      this.originTarget = [1, 2, 3]
    }
  },
  components: {
    HelloWorld,
    hostsPanel
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
