<template>
  <el-tabs type="border-card" v-model="activeName" @tab-click="handleClick">
    <el-tab-pane label="部门管理" name="DepMara"><DepMara></DepMara></el-tab-pane>
    <el-tab-pane label="奖惩规则" name="EcMana"><EcMana></EcMana></el-tab-pane>
    <el-tab-pane label="职称管理" name="JoblevelMana"><JoblevelMana></JoblevelMana></el-tab-pane>
    <el-tab-pane label="权限组" name="PermissMana"><PermissMana></PermissMana></el-tab-pane>
    <el-tab-pane label="职位管理" name="PosMana"><PosMana></PosMana></el-tab-pane>
  </el-tabs>
</template>

<script>
  import DepMara from '../../components/sys/basic/DepMana.vue'
  import EcMana from '../../components/sys/basic/EcMana.vue'
  import JoblevelMana from '../../components/sys/basic/JoblevelMana.vue'
  import PermissMana from '../../components/sys/basic/PermissMana.vue'
  import PosMana from '../../components/sys/basic/PosMana.vue'
  export default {
    name: "SysBasic",
     data() {
      return {
        activeName: 'PosMana'
      };
    },
    methods: {
      handleClick(tab, event) {
        console.log(tab, event);
      }
    },
    components:{
      DepMara,
      EcMana,
      JoblevelMana,
      PermissMana,
      PosMana
    }
  }
  
</script>

<style scoped>

</style>
