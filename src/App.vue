<template>
  <div class="common-layout" style="height: 100%">
    <el-container style="height: 100%">
      <el-header>Header</el-header>
      <el-container>
        <el-aside width="400px">
          <el-container class="aside-box">
            <el-row>
              <el-check-tag
                class="relTypeButton"
                size="mini"
                v-for="item in relTypes"
                :key="item.name"
                @change="onChange(item.id)"
                :checked="item.use"
                >{{ item.name }}</el-check-tag
              >
            </el-row>
          </el-container>
          <el-container class="aside-box">
            <el-switch
              v-model="fixNodes"
              active-text="锁定布局"
              inactive-text="流动布局"
              @change="holdFixChange"
            >
            </el-switch>
          </el-container>
          <el-container class="aside-box">
            <el-row>
              <el-select
                v-model="node1Name"
                filterable
                placeholder="实体1"
                @change="selectNode1"
                class="select-box"
              >
                <el-option
                  v-for="item in options"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                >
                </el-option>
              </el-select>
              <el-select
                v-model="node2Name"
                filterable
                placeholder="实体2"
                @change="selectNode2"
                class="select-box"
              >
                <el-option
                  v-for="item in options"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                >
                </el-option>
              </el-select>
              <el-button
                class="select-button"
                type="success"
                @click="mergeNodes"
                >合并</el-button
              >
              <el-select
                v-model="relTypeNow"
                filterable
                placeholder="关系类型"
                class="select-box"
              >
                <el-option
                  v-for="item in relIdMap"
                  :key="item"
                  :label="item"
                  :value="item"
                >
                </el-option>
              </el-select>
              <el-select
                v-model="relValueNow"
                filterable
                placeholder="关系权重"
                class="select-box"
              >
                <el-option
                  v-for="item in relValues"
                  :key="item"
                  :label="item"
                  :value="item"
                >
                </el-option>
              </el-select>
              <el-button class="select-button" type="success" @click="addRel"
                >新建</el-button
              >
            </el-row>
          </el-container>
        </el-aside>
        <el-main>
          <div id="main"></div>
        </el-main>
      </el-container>
    </el-container>
  </div>
  <!-- <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/> -->
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import * as echarts from "echarts";
import { ref } from "vue";
export default {
  name: "App",
  data() {
    return {
      value1: true,
      graph: [],
      nodes: [],
      filtedLinks: [],
      filtedNodes: [],
      links: [],
      node1: {},
      node2: {},
      chart: {},
      option: {},
      relValues: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
      relIdMap: [
        "Cause-Effect",
        "Instrument-Agency",
        "Message-Topic",
        "Product-Producer",
        "Member-Collection",
        "Entity-Origin",
        "Entity-Destination",
        "Component-Whole",
        "Content-Container",
      ],
      relTypes: [
        {
          name: "Cause-Effect",
          color: "#626aef",
          use: true,
          id: 0,
        },
        {
          name: "Instrument-Agency",
          color: "#626aef",
          use: true,
          id: 1,
        },
        {
          name: "Message-Topic",
          color: "#626aef",
          use: true,
          id: 2,
        },
        {
          name: "Product-Producer",
          color: "#626aef",
          use: true,
          id: 3,
        },
        {
          name: "Member-Collection",
          color: "#626aef",
          use: true,
          id: 4,
        },
        {
          name: "Entity-Origin",
          color: "#626aef",
          use: true,
          id: 5,
        },
        {
          name: "Entity-Destination",
          color: "#626aef",
          use: true,
          id: 6,
        },
        {
          name: "Component-Whole",
          color: "#626aef",
          use: true,
          id: 7,
        },
        {
          name: "Content-Container",
          color: "#626aef",
          use: true,
          id: 8,
        },
      ],
      history: [],
      options: ref([]),
      node1Name: ref(""),
      node2Name: ref(""),
      relTypeNow: ref(""),
      relValueNow: ref(""),
      fixNodes: false,
      nodeSwitch: 1,
    };
  },
  created() {},
  components: {
    // HelloWorld
  },
  mounted() {
    this.initData();
    this.initGraph();
  },
  methods: {
    initData() {
      var graph = require("./datas/echart_use_data_predict.json");
      this.graph = graph;
      this.nodes = graph.nodes;
      this.links = graph.links;
      for (var node of this.nodes) {
        var option = {
          value: node.id,
          label: node.name,
        };
        this.options.push(option);
      }
    },
    initGraph() {
      let main = echarts.init(document.getElementById("main"), "dark"); // 一键切换神色模式，有点给劲哦
      this.chart = main;
      this.graph.nodes.forEach(function (node) {
        node.label = {
          show: node.symbolSize > 13,
        };
      });
      let option = {
        title: {
          text: "实体与关系",
          subtext: "Default layout",
          top: "bottom",
          left: "right",
        },
        tooltip: {},
        legend: [
          {
            // selectedMode: 'single',
            data: this.graph.categories.map(function (a) {
              return a.name;
            }),
          },
        ],
        animationDuration: 1500,
        animationEasingUpdate: "quinticInOut",
        series: [
          {
            labelLayout: {
              hideOverlap: true, // 标签重叠自动隐藏
            },
            draggable: false,
            edgeSymbol: ["circle", "arrow"], // 设置箭头
            edgeSymbolSize: [1, 6], // 设置箭头的大小
            name: "实体与关系",
            type: "graph",
            layout: "force",
            force: {
              repulsion: 200,
              gravity: 0.1,
              edgeLength: [120, 200],
            },
            data: this.filtedNodes,
            links: this.filtedLinks,
            categories: this.graph.categories,
            roam: true,
            label: {
              position: "right",
              formatter: "{b}",
              show: true, // 显示标签文字
            },
            lineStyle: {
              color: "source",
              curveness: 0.3,
            },
            emphasis: {
              focus: "adjacency",
              lineStyle: {
                width: 10,
              },
            },
          },
        ],
      };
      this.option = option;
      var _this = this; // 让响应函数也能访问到vue
      this.filter();
      main.setOption(option);

      // 响应函数
      main.on("click", { dataType: "node" }, function (data) {
        console.log(data);
        console.log(this.nodeSwitch)
        if (_this.nodeSwitch == 1) {
          _this.node1Name = data.data.name;
          _this.node1.id = data.data.id;
          _this.nodeSwitch = 2;
        } else {
          _this.node2Name = data.data.name;
          _this.node2.id = data.data.id;
          _this.nodeSwitch = 1;
        }
      });
      main.on("dblclick", { dataType: "edge" }, function (data) {
        console.log(data);
        for (var i = 0; i < _this.links.length; i++) {
          if (data.data.source == _this.links[i].source) {
            if (data.data.target == _this.links[i].target) {
              if (data.data.value == _this.links[i].value) {
                _this.links.splice(i, 1);
                break;
              }
            }
          }
        }
        _this.updateGraph();
      });
    },
    updateGraph() {
      // this.chart.setOption({
      //   series:[{
      //     data: this.nodes,
      //     links: this.links
      //   }]
      // })
      this.filter();
      let main = echarts.init(document.getElementById("main"), "dark"); // 不懂这样为社么比注释掉的方法更快，但是确实更快
      main.setOption(this.option);
    },
    filter() {
      this.filtedLinks.splice(0);
      this.filtedNodes.splice(0);

      for (let node of this.nodes) {
        if (this.fixNodes) {
          node.fixed = true;
        } else {
          node.fixed = false;
        }
        this.filtedNodes.push(node);
      }
      for (let link of this.links) {
        console.log(this.relTypes[this.relIdMap.indexOf(link.value)].use);
        if (this.relTypes[this.relIdMap.indexOf(link.value)].use) {
          this.filtedLinks.push(link);
        }
      }
    },
    onChange(id) {
      console.log(id);
      if (this.relTypes[id].use == true) {
        this.relTypes[id].use = false;
      } else {
        this.relTypes[id].use = true;
      }
      this.updateGraph();
    },
    mergeNodes() {
      // 合并两个节点
      console.log(this.node1.id, this.node2.id);
      this.history.push((this.node1.id, this.node2.id));

      // 合并节点的关系，重复的关系线要变粗
      var map1 = new Map();
      for (let i = 0; i < this.links.length; i++) {
        if (this.links[i].source == this.node1.id) {
          this.links[i].source = this.node2.id;
          console.log(this.links[i].source);
        } else if (this.links[i].target == this.node1.id) {
          this.links[i].target = this.node2.id;
          console.log(this.links[i].target);
        }
        var label =
          this.links[i].source +
          "@@@" +
          this.links[i].target +
          "@@@" +
          this.links[i].value;
        if (map1.has(label)) {
          map1.set(label, map1.get(label) + this.links[i].lineStyle.width);
        } else {
          map1.set(label, this.links[i].lineStyle.width);
        }
      }
      this.links.splice(0);
      for (let temp of map1) {
        var a = temp[0].split("@@@");
        let s = a[0];
        let t = a[1];
        let c = a[2];
        let link = {
          source: s,
          target: t,
          value: c,
          lineStyle: {
            width: temp[1],
          },
          labe: {
            formatter: "{c}",
          },
        };
        this.links.push(link);
      }

      console.log("aaaaa", this.links);

      // 删除节点,更新节点大小
      var node1Size = 0;
      for (let i = 0; i < this.nodes.length; i++) {
        if (this.nodes[i].id == this.node1.id) {
          node1Size = this.nodes[i].symbolSize;
          this.nodes.splice(i, 1);
          break;
        }
      }
      for (let i = 0; i < this.nodes.length; i++) {
        if (this.nodes[i].id == this.node2.id) {
          this.nodes[i].symbolSize = this.nodes[i].symbolSize + node1Size;
          break;
        }
      }

      // 删除option中对应节点
      for (var i = 0; i < this.options.length; i++) {
        if (this.options[i].value == this.node1.id) {
          console.log(this.options[i]);
          this.options.splice(i, 1);
          console.log(this.options[i]);
          this.node1.id = -1;
          this.value = "";
          break;
        }
      }
      this.updateGraph();
    },
    selectNode1(v) {
      this.node1.id = v;
    },
    selectNode2(v) {
      this.node2.id = v;
    },
    addRel() {
      let newLink = {
        source: this.node1.id,
        target: this.node2.id,
        value: this.relTypeNow,
        lineStyle: {
          width: this.relValueNow,
        },
        labe: {
          formatter: "{c}",
        },
      };
      this.links.push(newLink);
      this.updateGraph();
    },
    holdFixChange(value) {
      this.fixNodes = value;
      console.log(this.fixNodes);
      this.updateGraph();
    },
  },
};
</script>

<style>
html {
  height: 100%;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100%;
}
#main {
  width: 100%;
  height: 100%;
}
#joinWord {
  font-size: 5px;
}
.el-header,
.el-footer {
  background-color: #b3c0d1;
  color: var(--el-text-color-primary);
  text-align: center;
  line-height: 60px;
}

.el-aside {
  background-color: #d3dce6;
  color: var(--el-text-color-primary);
  text-align: center;
  line-height: 100px;
}

.el-main {
  background-color: #e9eef3;
  color: var(--el-text-color-primary);
  padding: 0px !important;
  text-align: center;
  line-height: 160px;
}

.aside-box {
  padding: 15px;
}

.relTypeButton {
  margin-left: 5px !important;
  margin-bottom: 5px !important;
}

.select-box {
  width: 140px;
  margin-left: 5px;
  margin-bottom: 5px;
}

.select-button {
  margin-left: 5px !important;
  margin-bottom: 5px !important;
}

body {
  margin: 0px;
  height: 100%;
}

body > .el-container {
  margin-bottom: 40px;
}
</style>
