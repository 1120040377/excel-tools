<!------------------------------------------- template ------------------------------------------->
<template>
  <div>
    <el-upload
      drag
      :before-upload="actionUpload"
    >
    <!-- multiple -->
      <el-icon class="el-icon--upload"><upload-filled /></el-icon>
      <div class="el-upload__text">拖拽文件到此处 OR <em>点击上传</em></div>
      <template #tip>
        <div class="el-upload__tip">上传一个文件为拆分/上传多个文件为合并</div>
      </template>
    </el-upload>
  </div>
</template>

<!-------------------------------------------- script -------------------------------------------->
<script>
export default {
  name: '',
  components: {
  },

  data () {
    return {
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
    actionUpload(file) {
      this.$emit('on-add', file)
      return false;
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

<!-------------------------------------------- style -------------------------------------------->
<style scoped lang="scss">

</style>
