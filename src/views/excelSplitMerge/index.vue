<!------------------------------------------- template ------------------------------------------->
<template>
  <div class="split-merge">
    <div class="split-merge-left" v-if="cacheFileList.length > 1">
      <el-tabs tab-position="left" style="height: 100%" class="demo-tabs">
        <el-tab-pane
          v-for="file in cacheFileList"
          :key="file.uid"
          :label="file.name"></el-tab-pane>
      </el-tabs>
    </div>
    <div class="split-merge-right">
      <!-- 操作栏 -->
      <div class="mb-5">
        <el-button @click="cacheFileList = []">重置</el-button>
      </div>
      
      <UploadFile
        v-if="type === 'upload'"
        @on-add="actionAddFile"
      ></UploadFile>
      
      <SplitFile
        v-if="type === 'split' && curWorkBook"
        :data="curWorkBook"
      ></SplitFile>
      <div v-if="type === 'merge'">合并模式</div>
    </div>
  </div>
</template>

<!-------------------------------------------- script -------------------------------------------->
<script>
import UploadFile from './components/uploadFile.vue'
import SplitFile from './components/splitFile.vue'
import * as XLSX from 'xlsx/xlsx.mjs';
export default {
  name: "",
  components: {
    UploadFile,
    SplitFile,
  },

  data() {
    return {
      type: "upload", // upload split merge
      cacheFileList: [],
      wbs: {},
      curFile: null,
      curWorkBook: null,
    };
  },

  /* 一.生命周期函数 */
  created() {},

  /* 二.监控函数 */
  watch: {
    cacheFileList: {
      deep: true,
      handler() {
        switch (this.cacheFileList.length) {
          case 1:
            this.chooseFile(0)
            this.type = "split";
            break;
          case 0:
            this.type = "upload";
            break;
          default:
            this.type = "merge";
            break;
        }
      },
    },
  },

  computed: {},

  /* 三.内部功能函数 */
  methods: {
    /* ----------------------事件调用函数------------------------ */
    // 事件调用函数注释
    async actionAddFile(file) {
      const data = await file.arrayBuffer();
      const workbook = XLSX.read(data);
      this.wbs[file.uid] = {
        name: file.name,
        uid: file.uid,
        workbook
      }
      this.cacheFileList.push(file);
      return false;
    },

    /* ----------------------内部功能函数------------------------ */
    // 选择文件
    chooseFile(index) {
      this.curFile = this.cacheFileList[index]
      this.curWorkBook = this.wbs[this.curFile.uid]
    },

    /* ----------------------服务请求函数------------------------ */
    // 服务请求函数注释
    ajaxXxx() {},
  },
};
</script>

<!-------------------------------------------- style -------------------------------------------->
<style scoped lang="scss">
.split-merge{
  width: 100%;
  height: 100%;
  display: flex;
  &-left{
    flex: 0 0 auto;
  }
  &-right{
    flex: 1;
    overflow: hidden;
  }
}
</style>
