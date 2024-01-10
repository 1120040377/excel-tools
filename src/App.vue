<template>
  <div class="page">
    <el-menu
      default-active="2"
      class="page-nav"
      :collapse="isCollapse"
    >
      <el-sub-menu index="1">
        <template #title>
          <el-icon><location /></el-icon>
          <span>Navigator One</span>
        </template>
        <el-menu-item-group title="Group One">
          <el-menu-item index="1-1">item one</el-menu-item>
          <el-menu-item index="1-2">item two</el-menu-item>
        </el-menu-item-group>
        <el-menu-item-group title="Group Two">
          <el-menu-item index="1-3">item three</el-menu-item>
        </el-menu-item-group>
        <el-sub-menu index="1-4">
          <template #title>item four</template>
          <el-menu-item index="1-4-1">item one</el-menu-item>
        </el-sub-menu>
      </el-sub-menu>
      <el-menu-item index="2">
        <el-icon><DocumentCopy /></el-icon>
        <span>Navigator Two</span>
      </el-menu-item>
    </el-menu>
    <div class="page-container">
      <div class="page-header">
        <el-icon class="collapse-btn" @click="isCollapse = !isCollapse"><Expand /></el-icon>
        <div>郭郭</div>
      </div>
      <div class="page-body">
        <RouterView />
      </div>
    </div>
  </div>
</template>

<script>
import { RouterView } from "vue-router";
export default {
  name: '',
  components: {
    RouterView
  },

  data () {
    return {
      isCollapse: true
    }
  },

  /* 一.生命周期函数 */
  created () {
  },

  /* 二.监控函数 */
  watch: {
  },

  computed: {
  },

  /* 三.内部功能函数 */
  methods: {
    /* ----------------------事件调用函数------------------------ */
    // 事件调用函数注释
    actionXxx () {

    },

    /* ----------------------内部功能函数------------------------ */
    // 内部功能函数注释
    doSomething () {

    },

    /* ----------------------服务请求函数------------------------ */
    // 服务请求函数注释
    ajaxXxx () {

    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>
<style scoped lang="scss">
.page {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  &-nav {
    flex: 0 0 auto;
  }
  &-container {
    flex: 1;
    display: flex;
    overflow: hidden;
    flex-direction: column;
  }
  &-header {
    padding: 16px;
    display: flex;
    align-items: center;
    height: 60px;
    // border-bottom: 1px #efefef;
    box-shadow: 0 1px 2px #efefef;
  }
  &-body {
    padding: 16px;
    flex: 1;
    background: #efefef;
  }
}
.collapse-btn{
  margin-right: 16px;
  cursor: pointer;
  transition: all .3s linear;
  &:hover{
    color: #3185f2;
  }
}
</style>
