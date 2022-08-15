<template>
  <div>
    excel
    <el-upload
      class="upload-btn"
      ref="upload"
      action
      accept=".xls, .xlsx"
      :show-file-list="false"
      :on-change="readExcel"
      :auto-upload="false"
    >
      <el-button
        slot="trigger"
        icon="el-icon-upload"
        size="small"
        type="primary"
        >导入</el-button
      >
    </el-upload>
    <el-button
      type="success"
      @click="outExcel"
      size="small"
      icon="el-icon-upload2"
      >导出1</el-button
    >
  </div>
</template>

<script>
import XLSX from 'xlsx'
import fs from 'file-saver'
export default {
  data() {
    return {
      jsonData: [
        {
          id: '1',
          name: '小智',
          age: 18,
          classes: '商学院',
        },
        {
          id: '2',
          name: '小白',
          age: 19,
          classes: '商学院',
        },
        {
          id: '3',
          name: '小蓝',
          age: 12,
          classes: '商学院',
        },
        {
          id: '4',
          name: '小花',
          age: 14,
          classes: '商学院',
        },
        {
          id: '5',
          name: '小粉',
          age: 15,
          classes: '商学院',
        },
        {
          id: '6',
          name: '小黄',
          age: 16,
          classes: '商学院',
        },
        {
          id: '7',
          name: '小红',
          age: 17,
          classes: '商学院',
        },
        {
          id: '8',
          name: '小黑',
          age: 19,
          classes: '商学院',
        },
      ],
      listHander: {
        id: '学号',
        name: '姓名',
        age: '年龄',
        classes: '学院',
      },
    }
  },

  methods: {
    readExcel(file) {
      // console.log('file', file);
      const types = file.name.slice(file.name.lastIndexOf('.'))
      const fileType = ['.xlsx', '.xls'].some((item) => item === types)
      // console.log(file.name.lastIndexOf('.'), types, fileType);
      //  校验格式
      if (!fileType) {
        this.$message('格式错误！请重新上传')
        return
      }
      // 返回workbook
      this.file2Xce(file).then((tabJson) => {
        console.log('tabJson', tabJson)
        // this.handleImportItem(tabJson) //处理导入的数据使导入的数据在页面中展示。这个方法根据自己的需求，属于定制化
      })
    },
    outExcel() {
      // this.jsonData是要导出的数据内容（表格里的内容），
      // this.listHander对应要导出内容的表头
      // 学生：指向的是excel文件名
      this.xlsx(this.jsonData, this.listHander, '学生')
    },

    // 读表单，返回workbook
    file2Xce(file) {
      return new Promise((resolve) => {
        const reader = new FileReader()
        reader.onload = (e) => {
          const data = e.target.result
          // 二进制流方式读取得到整份Excel表格对象
          this.excelData = XLSX.read(data, {
            type: 'binary',
          })
          // console.log('exvelData', this.excelData);
          // 只取第一个工作表
          const wsname = this.excelData.SheetNames[0] // 取第一张表
          const ws = XLSX.utils.sheet_to_json(this.excelData.Sheets[wsname]) // 生成json表格内容
          resolve(ws)
        }
        // 以二进制方式打开文件
        reader.readAsBinaryString(file.raw) //file.raw取上传文件的File
      })
    },

    xlsx(json, fields, filename = '.xlsx') {
      const s2ab = (s) => {
        var buf
        if (typeof ArrayBuffer !== 'undefined') {
          buf = new ArrayBuffer(s.length)
          var view = new Uint8Array(buf)
          for (var i = 0; i != s.length; ++i) view[i] = s.charCodeAt(i) & 0xff
          return buf
        } else {
          buf = new Array(s.length)
          for (var i = 0; i != s.length; ++i) buf[i] = s.charCodeAt(i) & 0xff
          return buf
        }
      }
      //导出xlsx
      json.forEach((item) => {
        for (let i in item) {
          if (fields.hasOwnProperty(i)) {
            item[fields[i]] = item[i]
          }
          delete item[i] //删除原先的对象属性
        }
      })
      let sheetName = filename //excel的文件名称
      let wb = XLSX.utils.book_new() //工作簿对象包含一SheetNames数组，以及一个表对象映射表名称到表对象。XLSX.utils.book_new实用函数创建一个新的工作簿对象。
      let ws = XLSX.utils.json_to_sheet(json, { header: Object.values(fields) }) //将JS对象数组转换为工作表。
      wb.SheetNames.push(sheetName)
      wb.Sheets[sheetName] = ws
      const defaultCellStyle = {
        font: { name: 'Verdana', sz: 13, color: 'FF00FF88' },
        fill: { fgColor: { rgb: 'FFFFAA00' } },
      } //设置表格的样式
      let wopts = {
        bookType: 'xlsx',
        bookSST: false,
        type: 'binary',
        cellStyles: true,
        defaultCellStyle: defaultCellStyle,
        showGridLines: false,
      } //写入的样式
      let wbout = XLSX.write(wb, wopts)
      let blob = new Blob([s2ab(wbout)], { type: 'application/octet-stream' })
      fs.saveAs(blob, filename + '.xlsx')
    },
  },
}
</script>

<style lang="" scoped></style>
