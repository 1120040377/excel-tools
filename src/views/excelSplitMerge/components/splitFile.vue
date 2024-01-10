<!------------------------------------------- template ------------------------------------------->
<template>
  <div>
    <el-card class="box-card">
      <el-tabs
        v-model="activeSheet"
        type="card"
        @tab-click="handleClick"
      >
        <el-tab-pane
          v-for="sheet in configList"
          :name="sheet.id"
          :label="sheet.sheetName"
          :key="sheet.id"
          ></el-tab-pane>
      </el-tabs>

      <!-- 拆分工作区 -->
      <div>
        <el-form label-width="100px">
          <!-- 首先选择拆分方式 -->
          <el-form-item label="拆分方式">
            <el-radio-group v-model="getCurSheet.mode">
              <el-radio label="copy">复制</el-radio>
              <el-radio label="split">拆分</el-radio>
            </el-radio-group>
          </el-form-item>

          <!-- 确定为拆分模式时，需要选择拆行列 -->
          <template v-if="getCurSheet.mode === 'split'">
            <el-form-item label="拆分规则">
              <div class="overflow-x-auto w-full">
                <el-radio-group class="flex flex-nowrap">
                  <div class="serial-no">序号</div>
                  <el-radio-button
                    class="header-radio-button"
                    v-for="(item, index) in getHeader[0].length"
                    disabled
                    :label="numberToLetter(item)"
                    :key="index"
                  />
                </el-radio-group>
                <el-radio-group class="flex flex-nowrap" v-model="getCurSheet.splitKey" v-for="(row, rowIndex) in getHeader" :key="`row${rowIndex}`">
                  <div class="serial-no">{{rowIndex + 1}}</div>
                  <el-radio-button
                    class="header-radio-button"
                    v-for="(item, index) in row"
                    :label="`${rowIndex}-${index}`"
                    :model-value="index"
                    :key="index"
                    :disabled="item === undefined"
                  >
                    {{ item === undefined ? '空' : item }}
                  </el-radio-button>
                </el-radio-group>
              </div>
            </el-form-item>
          </template>
        </el-form>
      </div>
      {{ getCurSheet }}

      <el-button class="split-download-btn" @click="actionDownload">下载</el-button>
    </el-card>
  </div>
</template>

<!-------------------------------------------- script -------------------------------------------->
<script>
import * as XLSX from 'xlsx/xlsx.mjs';
import { ElMessage } from 'element-plus'
export default {
  name: "",
  components: {},

  props: {
    data: {
      type: Object,
      default: () => {},
    },
  },

  data() {
    return {
      activeSheet: "",
      configList: [],
    };
  },

  /* 一.生命周期函数 */
  created() {
    this.initData();
  },

  /* 二.监控函数 */
  watch: {},

  computed: {
    getCurSheet () {
      return this.configList.find(item => item.id === this.activeSheet)
    },
    getHeader () {
      const { Sheets } = this.data.workbook;
      const { sheetName } = this.getCurSheet
      const worksheet = Sheets[sheetName];

      const sheetJson = XLSX.utils.sheet_to_json(worksheet, { header: 1 })
      return sheetJson.slice(0, 10)
    }
  },

  /* 三.内部功能函数 */
  methods: {
    /* ----------------------事件调用函数------------------------ */
    // 下载拆分文件
    actionDownload() {
      const fileMap = {}
      const copySheets = []

      this.configList.filter(config => {
        const { mode, sheetName } = config
        if (mode !== 'copy') return true

        // 将仅复制的 Sheet 放到一个数组中
        copySheets.push({
          sheetName,
          worksheet: this.data.workbook.Sheets[sheetName]
        })
      }).forEach(config => {
        // 需要拆分的 Sheet 调用方法来处理
        this.genSheet(config, fileMap)
      })

      Object.values(fileMap).forEach(file => {
        const workbook = XLSX.utils.book_new();

        [...file.Sheets, ...copySheets].forEach(o => {
          XLSX.utils.book_append_sheet(workbook, o.worksheet, o.sheetName);
        })
        
        XLSX.writeFile(workbook, `${file.fileName}-${this.data.name}`);
      })

      console.log('fileMap', fileMap)
    },

    /* ----------------------内部功能函数------------------------ */
    // 内部功能函数注释
    initData() {
      const { SheetNames } = this.data.workbook;
      console.log(this.data.workbook)

      this.configList = SheetNames.map((sheetName) => {
        return {
          id: Math.random().toString(32).slice(2),
          mode: 'copy', // copy 复制 split 拆分
          sheetName,
          splitKey: "", // 基于什么字段拆分
        };
      });

      this.activeSheet = this.configList[0].id;
      console.log(this.data.workbook, this.configList);
    },
    // 生成 Sheet
    genSheet (sheetConfig, fileMap) {
      const { Sheets } = this.data.workbook;
      const { sheetName, splitKey, mode } = sheetConfig
      const [rowIndex, colIndex] = splitKey.split('-').map(i => Number(i))

      if (mode === 'split' && splitKey === '') return ElMessage.warning('请选择拆分规则')
      const worksheet = Sheets[sheetName];

      // 获取每个 Sheet 的标题栏部分，拆分多个 Sheet 的时候需要用
      const headerRange = { s: { r: 0, c:0 }, e: { r: rowIndex, c: 1000 } };
      const headerRows = XLSX.utils.sheet_to_json(worksheet, { range: headerRange, header: 1 });

      const maxRow = Number(/\d+/.exec(worksheet['!ref'].split(':')[1])[0])


      // 获取其他的数据
      const dataRange = { s: { r: rowIndex + 1, c:0 }, e: { r: maxRow, c: 1000 } };
      // filter 是为了过滤空行，通常来说，拆分数据的时候不需要保留空行
      const dataRows = XLSX.utils.sheet_to_json(worksheet, { range: dataRange, header: 1 }).filter(item => item.length);

      const sheetDataMap = {}
      // 将单个 Sheet 分组
      dataRows.forEach(row => {
        const key = row[colIndex]
        if (sheetDataMap[key]) {
          sheetDataMap[key].push(row)
        } else {
          sheetDataMap[key] = [row]
        }
      })

      Object.entries(sheetDataMap).forEach(item => {
        const [fileName, sheetData] = item
        if (!fileMap[fileName]) {
          fileMap[fileName] = {
            fileName,
            Sheets: []
          }
        }

        const newWorksheet = XLSX.utils.aoa_to_sheet([...headerRows, ...sheetData])
        newWorksheet['!merges'] = worksheet['!merges']
        fileMap[fileName].Sheets.push({
          sheetName,
          worksheet: newWorksheet
        })
      })
    },

    numberToLetter(number) {
      var dividend = number;
      var letter = "";

      while (dividend > 0) {
        var modulo = (dividend - 1) % 26;
        letter = String.fromCharCode(65 + modulo) + letter;
        dividend = Math.floor((dividend - 1) / 26);
      }

      return letter;
    },

    /* ----------------------服务请求函数------------------------ */
    // 服务请求函数注释
    ajaxXxx() {},
  },
};
</script>

<!-------------------------------------------- style -------------------------------------------->
<style lang="scss">
.header-radio-button{
  .el-radio-button__inner{
    width: 100px;
    height: 32px;
  }
}
</style>
<style scoped lang="scss">
.serial-no{
  flex: 0 0 auto;
  width: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px 0 0 4px;
  border: var(--el-border);
  height: 32px;
  font-size: 14px;
  color: #999;
}
.split-download-btn{
  width: 200px;
  display: block;
  margin: 32px auto 0;
}
</style>
