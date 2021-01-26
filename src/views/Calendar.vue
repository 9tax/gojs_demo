<template>
  <div id="box">
    <!-- left -->
    <div class="left">
      <el-calendar @click="compile()">
        <!-- 这里使用的是 2.5 slot 语法，对于新项目请使用 2.6 slot 语法-->
        <template slot="dateCell" slot-scope="{ date, data }">
          <p>
            <!--这里原本有动态绑定的class，去掉-->
            {{
              data.day
                .split("-")
                .slice(1)
                .join("-")
            }}<br />
           
          </p>
          <p class="make_event"> {{ dealMyDate(data.day) }}</p>
        </template>
      </el-calendar>
    </div>
    <!-- right -->
    <div class="right">
      <div class="add_data">
        <p>添加日期:</p>
        <el-input
          class="right_inp1"
          v-model="input"
          placeholder="请输入日期"
          suffix-icon="el-icon-date"
        ></el-input>
      </div>
      <div class="add_event">
        <p>添加事件:</p>
        <el-input
          class="right_inp1"
          v-model="input"
          placeholder="请输入内容"
        ></el-input>
      </div>

      <el-button class="right_inp1" type="primary" plain>提交</el-button>
    </div>
  </div>
</template>

<script>
export default {
  name: "calendar",
  data() {
    return {
      resDate: [
        { date: "2020-12-29", content: "放假" },
        { date: "2020-12-31", content: "去交电费" },
        { date: "2021-01-06", content: "去学习vue" },
      ],
      input: "", // input的值
    };
  },
  created() {},
  computed: {},
  watch: {},
  mounted() {},
  methods: {
    dealMyDate(v) {
      // console.log(v)
      // console.log(typeof v)
      let len = this.resDate.length;
      let res = "";
      for (let i = 0; i < len; i++) {
        if (this.resDate[i].date == v) {
          res = this.resDate[i].content;
          break;
        }
      }
      return res;
    },
    compile() {
      console.log(1);
    },
  },
};
</script>

<style scoped>
>>> .el-calendar-tabel thead th{
    background: lawngreen;
}
#box {
  width: 100%;
  display: flex;
}
.left {
  width: 70%;
  /* background: palegreen; */
}
.is-selected {
  color: #1989fa;
}
.right {
  width: 27%;
  margin-top: 100px;
  /* background: palevioletred; */
}
/* .right_inp1 {
  margin-top: 10px;
} */
.add_data {
  width: 100%;
  height: 80px;
  display: flex;
  /* background: palevioletred; */
  line-height: 80px;
}
.add_data p {
  width: 60%;
  /* background: palegreen; */
}
.add_event {
  width: 100%;
  height: 80px;
  display: flex;
  /* background: palevioletred; */
  line-height: 80px;
}
.add_event p {
  width: 60%;
  /* background: palegreen; */
}
.make_event{
    color: lawngreen;
}

</style>
