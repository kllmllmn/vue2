<template>
  <div>
    <!-- TODO -->
    <el-button type="primary" size="small" class="file-wrap"
      ><a
        class="seat"
        download="批量导入联系人模板"
        href="http://localhost:8080/public/templateExcel/infoPush-batch-import.xlsx"
      ></a
      >文件模板下载</el-button
    >

    <el-button type="primary" size="small" class="file-wrap"
      ><input
        class="seat"
        type="file"
        @change="excel2Array($event)"
      />选择文件导入</el-button
    >
    <el-button @click="array2Excel(array2JsonData, 'test')">导出</el-button>
    <div id="table"></div>
  </div>
</template>

<script>
import XLSX from "xlsx";
export default {
  name: "ExeclImportExport",
  data() {
    return {
      execlData: [
        ["序号", "姓名", "年龄", "性别"],
        ["1", "张三", "22", "男"],
        ["2", "里斯", "23", "男"],
      ],
      counter: 1,
      // 对象数组转二维数组
      array2JsonData: [],
      objArr: [],
    };
  },
  computed: {
    bigCounter() {
      return this.counter + 1;
    },
  },
  methods: {
    excel2Array(evt, fn) {
      const keyMaps = {
        num: "序号",
        phone: "手机号",
        age: "年龄",
        name: "姓名",
        applyAge: "申请信息_年龄",
        address: "地址",
        duration: "工作信息_时间段",
      };
      const option = {
        header: 1,
        defval: "",
      };
      // console.log(evt, "evt");
      // console.log(fn, "fn");
      let _this = this;
      console.log("start analysis excel ");
      // elementui库里的loading
      let loading = this.$loading({ text: "请稍后..." });
      let file;
      let files = evt.target.files;

      if (!files || files.length == 0) return;

      file = files[0];

      let reader = new FileReader();
      reader.onload = function (e) {
        // console.log(e, "e");
        // pre-process data
        let binary = "";
        let bytes = new Uint8Array(e.target.result);
        // console.log(bytes, "bytes");

        // 这种写法可以省去for循环
        // let data = new Uint8Array(e.target.result);
        // let wb = XLSX.read(data, {type: 'array'});

        let length = bytes.byteLength;
        for (let i = 0; i < length; i++) {
          binary += String.fromCharCode(bytes[i]);
        }
        // console.log(binary, "binary");

        /* read workbook */
        let wb = XLSX.read(binary, { type: "binary" });
        // console.log(wb, "wb");
        /* grab first sheet */
        let wsname = wb.SheetNames[0];
        let ws = wb.Sheets[wsname];

        let jsonArray = XLSX.utils.sheet_to_json(ws, option);
        console.log("json:", jsonArray);

        jsonArray.forEach((item, index) => {
          if (index > 1) {
            const [num, phone, age, name, applyAge, address, , infoDuration] =
              item;
            // console.log(num, phone, age, name, applyAge, address, infoDuration);
            let workJsonData = { infoDuration };
            // console.log("workJsonData", workJsonData);
            _this.objArr.push({
              num,
              phone,
              age,
              name,
              applyAge,
              address,
              workJsonData,
            });
          }
        });
        console.log("objArr", _this.objArr);
        // console.log("key", _this.convertKeys(_this.objArr, keyMaps));

        /*  // 不是响应式
        _this.array2JsonData = jsonArray.map((item) => {
          Object.values(item);
        }); */
        // jsonArray.forEach((item) => {
        //   _this.array2JsonData.push(Object.values(item));
        // });
        // _this.array2JsonData.unshift(Object.keys(jsonArray[0])); // 第一行，表头
        // console.log("_this.array2JsonData", _this.array2JsonData);
        // jsonArray是从excel读出的数据
        // fn(jsonArray);
        loading.close();

        /* 可以调用这个方法生成HTML预览 */
        let HTML = XLSX.utils.sheet_to_html(ws);
        // console.log(HTML, "html");
        document.getElementById("table").insertAdjacentHTML("afterbegin", HTML);
      };
      // console.log("bigCounter", this.bigCounter);

      reader.readAsArrayBuffer(file);
    },
    array2Excel(arrayData, excelName) {
      // console.log("bigCounter", this.bigCounter);
      let loading = this.$loading({ text: "请稍后..." });
      let data = [["no data", "no data", "no data"]];
      if (!arrayData) {
        arrayData = data;
      }

      /* convert from array of arrays to workbook */
      let worksheet = XLSX.utils.aoa_to_sheet(arrayData);
      // var worksheet = XLSX.utils.json_to_sheet(data.data);

      /* generate workbook and add the worksheet */
      let new_workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(new_workbook, worksheet, "sheet1");

      /* save to file */
      XLSX.writeFile(new_workbook, excelName + ".xlsx");

      loading.close();
    },
    /**
     * 转换 key
     * @param excelData
     * @param keysMap
     */
    convertKeys(excelData, keysMap) {
      return excelData.map((excelItem) => {
        return Object.entries(excelItem).reduce((prev, curt) => {
          const [curtKey, curtValue] = curt;

          // 更新 key
          const mappedKey = keysMap[curtKey];
          if (mappedKey) {
            prev[mappedKey] = curtValue;
          } else {
            prev[curtKey] = curtValue;
          }

          return prev;
        }, {});
      });
    },
  },
};
</script>

<style scoped></style>
