<template>
  <div id="sample">
    <div style="width: 100%; display: flex; justify-content: space-between">
      <!-- 左侧 -->
      <div
        id="palette"
        style="width: 100px; height: 700px; margin-right: 2px; background-color: whitesmoke; border: solid 1px black"
      ></div>
      <!-- 中间 -->
      <div
        id="myDiagramDiv"
        ref="diag"
        v-on:changed-selection="changedSelection()"
        style="width:60%; height: 700px; border: solid 1px black"
      ></div>
      <!-- 右边 -->
      <div style="width:30%; height: 700px; border: solid 1px black">
        <div class="rig_top">
          <p>节点属性</p>
          <div class="right_top">
            <p class="right_name">选择节点ID</p>
            <el-input
              class="right_inp1"
              v-model="currentNodeText"
              :disabled="currentNode1 === null"
              placeholder="请填写选择节点名称"
            ></el-input>
          </div>
          <div class="right_cen">
            <p class="right_key">选择节点名称</p>
            <el-input
              class="right_inp2"
              v-model="currentNodeKey"
              :disabled="currentNode2 === null"
              placeholder="请填写选择节点ID"
            ></el-input>
          </div>
          <!-- 保存按钮 -->
          <div>
            <el-button type="primary" plain id="saveModel" @click="save()"
              >保存</el-button
            >
            <el-button type="primary" plain @click="load()">加载</el-button>
          </div>
        </div>

        <!-- 表格数据 -->
        <div class="tabel">
          <p class="tabel_p">规则列表</p>
          <gojsdata :proa="gojs">12</gojsdata>
          <!-- 表格 -->
          <!-- <template>
            <el-table :data="tableData" style="width: 100%">
              <el-table-column prop="date" label="id" width="180">
              </el-table-column>
              <el-table-column prop="name" label="条件" width="180">
              </el-table-column>
              <el-table-column prop="address" label="结果"> </el-table-column>
            </el-table>
          </template> -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import gojsdata from "../components/gojs/gojsdata";
// import gojsdata from "../utils/gojsdata";
import go from "gojs";
// import Figures from "../../node_modules/gojs/extensions/Figures"
let $ = go.GraphObject.make;
var red = "orangered"; // 0 or false
var green = "forestgreen"; // 1 or true
var myDiagram = null;

import "gojs/extensions/Figures.js";
export default {
  components: {
    gojsdata,
  },
  data() {
    return {
      diagramData: {
        class: "go.GraphLinksModel",
        linkFromPortIdProperty: "fromPort",
        linkToPortIdProperty: "toPort",
        nodeDataArray: [
          // {category: "input", key: "input1", loc: "-150 -80",text:"a",name:"豆"},
          // {category:"or", key:"or1", loc:"-70 0" },
          // {category:"not",key:"not1",loc:"10 0"},
          // {category:"xor",key:"xor1",loc:"100 0"},
          // {category:"or", key:"or2", loc:"200 0" },
          // {category:"output",key:"output1",loc:"200 -100"}
        ],
        linkDataArray: [
          // {from:"input1",fromPort:"out",to:"or1",toPort:"in1"},
          // {from:"or1",fromPort:"out",to:"not1",toPort:"in"},
          // {from:"not1",fromPort:"out",to:"or1",toPort:"in2"},
          // {from:"not1",fromPort:"out",to:"xor1",toPort:"in1"},
          // {from:"xor1",fromPort:"out",to:"or2",toPort:"in1"},
          // {from:"or2",fromPort:"out",to:"xor1",toPort:"in2"},
          // {from:"xor1",fromPort:"out",to:"output1",toPort:""}
        ],
      },
      savedModelText: "", // 用于双向绑定的
      currentNode1: null,
      currentNode2: null,
      gojs: {},//把生成的规则传到js文件解析中
    };
  },
  created() {
  },
  mounted() {

    myDiagram = $(
      go.Diagram,
      "myDiagramDiv", // 在HTML DIV元素“ myDiagramDiv”中创建一个新的Diagram
      {
        "draggingTool.isGridSnapEnabled": true, // 拖动的节点将捕捉到10x10单元格的网格
        "undoManager.isEnabled": true,
        ChangedSelection: (e) => {
          this.$emit("changed-selection", e);
          // 在这把模型修改过的值val，在附给input
          let node = e.diagram.selection.first();
          console.log(myDiagram.model);
          if (node instanceof window.go.Node) {
            // true
            this.currentNode1 = node;
            this.currentNode2 = node;
            this.currentNodeText = node.data.text;
            this.currentNodeKey = node.data.name;
            // console.log(node.data.key)
          } else {
            this.currentNode1 = null;
            this.currentNodeText = "";
            this.currentNode2 = null;
            this.currentNodeKey = "";
          }
        },
      }
    );
    // 为了拿到gojs封装的model这个方法
    this.pmodel = myDiagram.model;
    // 修改文档后，在标题上添加“ *”并启用“保存”按钮
    myDiagram.addDiagramListener("Modified", function(e) {
      e;
      var button = document.getElementById("saveModel");
      if (button) button.disabled = !myDiagram.isModified;
      var idx = document.title.indexOf("*");
      if (myDiagram.isModified) {
        if (idx < 0) document.title += "*";
      } else {
        if (idx >= 0) document.title = document.title.substr(0, idx);
      }
    });
    var palette = new go.Palette("palette"); // 在HTML DIV元素“ palette”中创建一个新的Palette
    //创建可重新链接的链接，这些链接将在可能的情况下避免越过节点，并跳过其路径中的其他链接
    myDiagram.linkTemplate = $(
      go.Link,
      {
        routing: go.Link.AvoidsNodes,
        curve: go.Link.JumpOver,
        corner: 3,
        relinkableFrom: true,
        relinkableTo: true,
        selectionAdorned: false, // 选中链接时不对其进行修饰，以使链接的颜色保持可见
        shadowOffset: new go.Point(0, 0),
        shadowBlur: 5,
        shadowColor: "blue",
      },
      new go.Binding("isShadowed", "isSelected").ofObject(),
      $(go.Shape, { name: "SHAPE", strokeWidth: 2, stroke: red })
    );

    // 为每种类型的节点定义模板
    var inputTemplate = $(
      go.Node,
      "Spot",
      this.nodeStyle(),
      $(go.Shape, "Circle", this.shapeStyle(), { fill: red }), // 将默认填充（从shapeStyle（））覆盖为红色
      $(
        go.Shape,
        "Rectangle",
        this.portStyle(false), // 唯一的port
        { portId: "", alignment: new go.Spot(1, 0.5) }
      ),
      $(
        go.TextBlock,
        { margin: 0, editable: true },
        new go.Binding("text", "text").makeTwoWay()
      ),
      $(
        go.TextBlock,
        { editable: true, alignment: new go.Spot(0.5, -0.2) },
        new go.Binding("text", "name").makeTwoWay()
      ),
      {
        // 如果双击，则输入节点将更改其值，以颜色表示。
        doubleClick: (e, obj) => {
          e.diagram.startTransaction("Toggle Input");
          var shp = obj.findObject("NODESHAPE");
          shp.fill = shp.fill === green ? red : green;
          this.updateStates();
          e.diagram.commitTransaction("Toggle Input");
        },
      }
    );

    var outputTemplate = $(
      go.Node,
      "Spot",
      this.nodeStyle(),
      $(go.Shape, "Rectangle", this.shapeStyle(), { fill: green }), // 覆盖默认填充（来自shapeStyle（））为绿色
      $(
        go.Shape,
        "Rectangle",
        this.portStyle(true), // 唯一的port
        { portId: "", alignment: new go.Spot(0, 0.5) }
      ),
      $(
        go.TextBlock,
        { margin: 0, editable: true },
        new go.Binding("text", "text").makeTwoWay()
      ),
      $(
        go.TextBlock,
        { editable: true, alignment: new go.Spot(0.5, -0.2) },
        new go.Binding("text", "name").makeTwoWay()
      )
    );

    var outputTemplate1 = $(
      go.Node,
      "Spot",
      this.nodeStyle(),
      $(go.Shape, "Rectangle", this.shapeStyle(), { fill: green }), // 覆盖默认填充（来自shapeStyle（））为绿色
      $(
        go.Shape,
        "Rectangle",
        this.portStyle(true), // 唯一的port
        { portId: "in", alignment: new go.Spot(0, 0.5) }
      ),
      $(
        go.Shape,
        "Rectangle",
        this.portStyle(false), // 唯一的port
        { portId: "out", alignment: new go.Spot(1, 0.5) }
      ),
      $(
        go.TextBlock,
        { margin: 0, editable: true },
        new go.Binding("text", "text").makeTwoWay()
      ),
      $(
        go.TextBlock,
        { editable: true, alignment: new go.Spot(0.5, -0.2) },
        new go.Binding("text", "name").makeTwoWay()
      )
    );

    var andTemplate = $(
      go.Node,
      "Spot",
      this.nodeStyle(),
      $(go.Shape, "AndGate", this.shapeStyle()),
      $(go.Shape, "Rectangle", this.portStyle(true), {
        portId: "in1",
        alignment: new go.Spot(0, 0.3),
      }),
      $(go.Shape, "Rectangle", this.portStyle(true), {
        portId: "in2",
        alignment: new go.Spot(0, 0.7),
      }),
      $(go.Shape, "Rectangle", this.portStyle(false), {
        portId: "out",
        alignment: new go.Spot(1, 0.5),
      }),
      $(
        go.TextBlock,
        { margin: 0, editable: true },
        new go.Binding("text", "text").makeTwoWay()
      ),
      $(
        go.TextBlock,
        { editable: true, alignment: new go.Spot(0.5, -0.2) },
        new go.Binding("text", "name").makeTwoWay()
      )
    );

    var orTemplate = $(
      go.Node,
      "Spot",
      this.nodeStyle(),
      $(go.Shape, "orgate", this.shapeStyle()),
      $(go.Shape, "Rectangle", this.portStyle(true), {
        portId: "in1",
        alignment: new go.Spot(0.16, 0.3),
      }),
      $(go.Shape, "Rectangle", this.portStyle(true), {
        portId: "in2",
        alignment: new go.Spot(0.16, 0.7),
      }),
      $(go.Shape, "Rectangle", this.portStyle(false), {
        portId: "out",
        alignment: new go.Spot(1, 0.5),
      }),
      $(
        go.TextBlock,
        { margin: 0, editable: true },
        new go.Binding("text").makeTwoWay()
      ),
      $(
        go.TextBlock,
        { margin: 0, editable: true },
        new go.Binding("text", "text").makeTwoWay()
      ),
      $(
        go.TextBlock,
        { editable: true, alignment: new go.Spot(0.5, -0.2) },
        new go.Binding("text", "name").makeTwoWay()
      )
    );

    // 将上面创建的模板添加到myDiagram和调色板
    myDiagram.nodeTemplateMap.add("input", inputTemplate);
    myDiagram.nodeTemplateMap.add("output", outputTemplate);
    myDiagram.nodeTemplateMap.add("output1", outputTemplate1);
    myDiagram.nodeTemplateMap.add("and", andTemplate);
    myDiagram.nodeTemplateMap.add("or", orTemplate);

    // 与调色板共享模板映射
    palette.nodeTemplateMap = myDiagram.nodeTemplateMap;

    palette.model.nodeDataArray = [
      { category: "input", text: "1", name: "name" },
      { category: "output", text: "2", name: "name" },
      { category: "output1", text: "3", name: "name" },
      { category: "and", text: "and", name: "" },
      { category: "or", text: "or", name: "" },
    ];

    // 加载初始图
    this.load();
    // 不断更新图标
    // this.loop();
    this.save(); // 保存按钮事件
  },
  computed: {
    /* 用于绑定input的值 */
    currentNodeText: {
      // 在这来获取到模型的值，赋值给input
      get: function() {
        let node = this.currentNode1;
        if (node instanceof window.go.Node) {
          return node.data.text;
        } else {
          return "";
        }
      },
      set: function(val) {
        // 修改input的值在赋值给模型
        let node = this.currentNode1;
        if (node instanceof window.go.Node) {
          myDiagram.model.startTransaction();
          myDiagram.model.setDataProperty(node.data, "text", val);
          myDiagram.model.commitTransaction("edited text");
        }
      },
    },
    currentNodeKey: {
      // 在这来获取到模型的值，赋值给input
      get: function() {
        let node = this.currentNode2;
        if (node instanceof window.go.Node) {
          return node.data.name;
        } else {
          return "";
        }
      },
      set: function(val) {
        // 修改input的值在赋值给模型
        let node = this.currentNode2;
        if (node instanceof window.go.Node) {
          myDiagram.model.startTransaction();
          myDiagram.model.setDataProperty(node.data, "name", val);
          myDiagram.model.commitTransaction("edited name");
        }
      },
    },
  },
  methods: {
    // 节点类型
    nodeStyle() {
      // 节点模板助手
      var sharedToolTip = $(
        "ToolTip",
        { "Border.figure": "RoundedRectangle" },
        $(
          go.TextBlock,
          { margin: 2 },
          new go.Binding("text", "", function(d) {
            return d.category;
          })
        )
      );

      return [
        new go.Binding("location", "loc", go.Point.parse).makeTwoWay(
          go.Point.stringify
        ),
        new go.Binding("isShadowed", "isSelected").ofObject(),
        {
          selectionAdorned: false,
          shadowOffset: new go.Point(0, 0),
          shadowBlur: 15,
          shadowColor: "blue",
          toolTip: sharedToolTip,
        },
      ];
    },

    //形状样式
    shapeStyle() {
      return {
        name: "NODESHAPE",
        fill: "lightgray",
        stroke: "darkslategray",
        desiredSize: new go.Size(40, 40),
        strokeWidth: 1,
      };
    },

    //前柱式构造
    portStyle(input) {
      return {
        desiredSize: new go.Size(6, 6),
        fill: "black",
        fromSpot: go.Spot.Right,
        fromLinkable: !input,
        toSpot: go.Spot.Left,
        toLinkable: input,
        toMaxLinks: 1,
        cursor: "pointer",
      };
    },
    // 根据其类型和输入值更新每个节点的值和外观
    updateStates() {
      var oldskip = this.myDiagram.skipsUndoManager;
      this.myDiagram.skipsUndoManager = true;
      // do all "input" nodes first
      this.myDiagram.nodes.each(function(node) {
        if (node.category === "input") {
          this.doInput(node);
        }
      });
      // 现在我们可以做所有其他类型的节点
      this.myDiagram.nodes.each(function(node) {
        switch (node.category) {
          case "and":
            this.doAnd(node);
            break;
          case "or":
            this.doOr(node);
            break;
          case "xor":
            this.doXor(node);
            break;
          case "not":
            this.doNot(node);
            break;
          case "nand":
            this.doNand(node);
            break;
          case "nor":
            this.doNor(node);
            break;
          case "xnor":
            this.doXnor(node);
            break;
          case "output":
            this.doOutput(node);
            break;
          case "input":
            break; // 上面已经调用了doInput
        }
      });
      this.myDiagram.skipsUndoManager = oldskip;
    },

    // helper predicate
    linkIsTrue(link) {
      // 假设给定的Link具有一个名为“ SHAPE”的Shape
      return link.findObject("SHAPE").stroke === green;
    },

    // 用于传播结果的帮助函数
    setOutputLinks(node, color) {
      node.findLinksOutOf().each(function(link) {
        link.findObject("SHAPE").stroke = color;
      });
    },
    //通过特定的功能更新节点的类型
    //根据传入的和节点类型确定从该节点发出的链接的颜色

    doInput(node) {
      // 输出只是该节点的Shape.fill
      this.setOutputLinks(node, node.findObject("NODESHAPE").fill);
    },

    doOutput(node) {
      //假设只有一个输入链接
      //我们只需要更新节点的Shape.fill
      node.linksConnected.each(function(link) {
        node.findObject("NODESHAPE").fill = link.findObject("SHAPE").stroke;
      });
    },
    doOutput1(node) {
      node.linksConnected.each(function(link) {
        node.findObject("NODESHAPE").fill = link.findObject("SHAPE").stroke;
      });
    },

    doOr(node) {
      var color = node.findLinksInto().any(this.linkIsTrue) ? green : red;
      this.setOutputLinks(node, color);
    },

    doXor(node) {
      var truecount = 0;
      node.findLinksInto().each(function(link) {
        if (this.linkIsTrue(link)) truecount++;
      });
      var color = truecount % 2 !== 0 ? green : red;
      this.setOutputLinks(node, color);
    },
    doXnor(node) {
      var truecount = 0;
      node.findLinksInto().each(function(link) {
        if (this.linkIsTrue(link)) truecount++;
      });
      var color = truecount % 2 === 0 ? green : red;
      this.setOutputLinks(node, color);
    },

    //updateStates
    // 每循环0.25秒更新一次图表this.$options.methods.loop.bind(this)()
    // loop: function() {
    //   setTimeout(function() {
    //     // this.$options.methods.updateStates();
    //     // this.$options.methods.loop();
    //   }, 250);
    // },

    // 将模型保存到JSON文本并从JSON文本加载模型，显示在Diagram
    save() {
      // document.getElementById("mySavedModel").value = myDiagram.model.toJson();
      myDiagram.isModified = false;
      // console.log(myDiagram);
      this.gojs = JSON.parse(myDiagram.model.toJson());
      console.log(myDiagram.model.toJson())
      console.log(this.gojs);
      if(this.gojs.linkDataArray.length>0){
        this.$message({
          message: '保存成功',
          type: 'success'
        });
      }
    },
    load() {
      var json1 = JSON.stringify(this.diagramData);
      myDiagram.model = go.Model.fromJson(json1);
      console.log(myDiagram.model);
    },

    // 规则解析
  },
};
</script>

<style scoped>
#sample {
  width: 100%;
  height: 100%;
  /* background: powderblue; */
}
.right_top {
  width: 100%;
  height: 70px;
  line-height: 70px;
  /* background: yellow; */
}
.right_cen {
  width: 100%;
  height: 70px;
  line-height: 70px;
  /* background: peru; */
}
.right_name {
  width: 40%;
  height: 70px;
  /* background: violet; */
  float: left;
}
.right_inp1 {
  width: 50%;
  height: 70px;
  /* background: turquoise; */
  float: left;
}
.right_key {
  width: 40%;
  height: 70px;
  /* background: violet; */
  float: left;
}
.right_inp2 {
  width: 50%;
  height: 70px;
  /* background: turquoise; */
  float: left;
}
.rig_top {
  width: 100%;
  height: 250px;
  border: 1px solid black;
  /* background: red; */
}
.tabel {
  width: 100%;
  height: 400px;
  margin-top: 5px;
  border: 1px solid black;
  /* background: purple; */
}
.tabel_p {
  width: 100%;
  line-height: 40px;
  height: 40px;
}
</style>
