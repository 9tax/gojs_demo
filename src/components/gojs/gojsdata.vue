<template>
  <div>
    <el-button class="but" @click="sheng()">生成规则</el-button>
    <!-- 表格 -->
      <el-table :data="tableData" style="width:100%; height:100">
        <el-table-column prop="id" label="id" width="130">
        </el-table-column>
        <el-table-column prop="conditions" label="条件" width="130">
        </el-table-column>
        <el-table-column prop="result" label="结果" width="130">
        </el-table-column>
        <el-table-column prop="address" label="处置策略" width="130">
          <a>点击添加</a>
        </el-table-column>
      </el-table>
    <!-- <div>{{proa}}</div> -->
  </div>
</template>

<script>
export default {
  name: "gojsdata",
  props: {
    proa: {
      type: Object,
    },
  }, // 父亲传过来的值
  data() {
    return {
      diagramData: {},
      nodeDataArray: [],
      linkDataArray: [],
      resList: [],
      orResList: [],
      andKeyList: [],
      orKeyList: [],
      andKeyInfoList: [],
      orKeyInfoList: [],
      tableData: [], // 表格数据
    };
  },
  created() {},
  mounted() {},
  methods: {
    sheng() {
      console.log(this.proa);
      this.getRule(this.proa);
      console.log(this.proa.nodeDataArray.category)
    },
    getAndKey(nodeData) {
      /**
       * 得到key为and的列表
       */
      if (nodeData["category"] === "and") {
        this.andKeyList.push(nodeData["key"]);
      }
    },

    getOrKey(nodeData) {
      /**
       * 得到key为or的列表
       */
      if (nodeData["category"] === "or") {
        this.orKeyList.push(nodeData["key"]);
      }
    },

    getAndKeyInfo(andKey, linkData) {
      /**
       * 得到与andKey有关系的信息
       */
      // console.log(linkData.from)
      if (linkData["to"] === andKey) {
        this.andKeyInfoList.splice(0, 0, linkData["from"]);
      }
      if (linkData["from"] === andKey) {
        this.andKeyInfoList.push(linkData["to"]);
      }
    },

    getOrKeyInfo(orKey, linkData) {
      /**
       * 得到与orKey有关系的信息
       */
      if (linkData["to"] === orKey) {
        this.orKeyInfoList.splice(0, 0, linkData["from"]);
      }
      if (linkData["from"] === orKey) {
        this.orKeyInfoList.push(linkData["to"]);
      }
    },

    orKeyInfoSplit(orKeyInfo) {
      /**
       * 将orKeyInfo信息进行分离
       */

      for (let i = 0; i <= orKeyInfo.length - 1; i++) {
        if (i !== orKeyInfo.length - 1) {
          var tempList = [];
          tempList.push(orKeyInfo[i]);
          tempList.push(orKeyInfo.slice(-1)[0]);
          this.orResList.push(tempList);
        }
      }
    },

    endListFunc(nodeKey) {
      for (var i in this.nodeDataArray) {
        if (this.nodeDataArray[i]["key"] === nodeKey) {
          return this.nodeDataArray[i]["text"];
        }
      }
    },

    result(xxx) {
      for (var nodeKeyList in xxx) {
        var res = xxx[nodeKeyList].map(this.endListFunc);

        this.resList.push(res.join(" "));
      }
    },

    tableDataFunc(resList) {
      for (let ruleIndex in resList) {
        let rule = resList[ruleIndex];
        if (rule.indexOf(" ") !== rule.lastIndexOf(" ")) {
          let ruleDict = {};
          ruleDict["id"] = Number(ruleIndex) + 1;
          let conditions = rule.slice(0, rule.lastIndexOf(" "));
          let newConditions = conditions.replace(/\s/, " AND ");
          ruleDict["conditions"] = newConditions;
          ruleDict["result"] = rule.slice(rule.lastIndexOf(" ") + 1);
          this.tableData.push(ruleDict);
        }

        if (rule.indexOf(" ") === rule.lastIndexOf(" ")) {
          let ruleDict = {};
          ruleDict["id"] = Number(ruleIndex) + 1;
          ruleDict["conditions"] = rule.slice(0, rule.lastIndexOf(" "));
          ruleDict["result"] = rule.slice(rule.lastIndexOf(" ") + 1);
          this.tableData.push(ruleDict);
        }
      }
    },

    getRule(row) {
      /**
       * 得到多组具有关系的规则列表
       */
      this.tableData = [];
      this.resList = [];
      console.log(row);
      this.nodeDataArray = row["nodeDataArray"];
      this.linkDataArray = row["linkDataArray"];
      console.log(this.nodeDataArray);
      console.log(this.linkDataArray);

      this.nodeDataArray.map(this.getAndKey);
      this.nodeDataArray.map(this.getOrKey);

      var allAndKInfo = [];
      this.andKeyList.map((item) => {
        this.linkDataArray.map((item1) => {
          this.getAndKeyInfo(item, item1);
        });
        allAndKInfo.push(JSON.parse(JSON.stringify(this.andKeyInfoList)));
        this.andKeyInfoList.length = 0;
      });

      var allOrKInfo = [];

      this.orKeyList.map((item) => {
        this.linkDataArray.map((item1) => {
          this.getOrKeyInfo(item, item1);
        });
        allOrKInfo.push(JSON.parse(JSON.stringify(this.orKeyInfoList)));
        this.orKeyInfoList.length = 0;
      });
      allOrKInfo.map(this.orKeyInfoSplit);
      this.result(allAndKInfo);
      this.result(this.orResList);
      console.log(this.resList);
      this.tableDataFunc(this.resList);
      console.log(this.tableData);
      this.nodeDataArray = [];
      this.linkDataArray = [];

      this.orResList = [];
      this.andKeyList = [];
      this.orKeyList = [];
      this.andKeyInfoList = [];
      this.orKeyInfoList = [];
    },
  },
};
</script>

<style scped>
.but {
  height: 40px;
  float: right;
}
</style>
