<template>
  <div class="main">
    <div id="canvasContainer"></div>
    <div class="buttons">
      <div class="button" @click="loadData">渲 染</div>
    </div>
  </div>
</template>

<script>
import G6 from "@antv/g6"; // 导入G6图引擎
import data from "./components/wordRelationshipData.json";
export default {
  name: 'WordRelationship',
  components: {},
  data() {
    return {
      dependenciesData: {}, // 可视化区域的数据（请求后保存在这里）
      words: [], // 从dependenciesData解析所得的【词项】，用于可视化渲染
      natures: [], // 从dependenciesData解析所得的【词性】，用于可视化渲染和数据修改
      natureTypes: [], // 从dependenciesData解析所得的【词性的类型】，用于元素链下拉菜单选择
      edges: [], // 从dependenciesData解析所得的【关系】，用于可视化渲染
    }
  },
  created() {
    console.log("page on create");
  },
  mounted() {
    console.log("page mounted");
  },
  methods: {
    // 加载数据
    loadData() {
      console.log("start load data");
      document.getElementById("canvasContainer").innerHTML = "";
      this.dependenciesData = data;
      console.log("finish load data");
      this.disposeData();
    },
    // 处理数据
    disposeData() {
      console.log("start dispose data");
      this.words = []; // 从dependenciesData解析所得的【词项】，用于可视化渲染
      this.natures = []; // 从dependenciesData解析所得的【词性】，用于可视化渲染和数据修改
      this.natureTypes = []; // 从dependenciesData解析所得的【词性的类型】，用于元素链下拉菜单选择
      this.edges = []; // 从dependenciesData解析所得的【关系】，用于可视化渲染

      let natureTypeTemp = []; // 声明临时数组用于在本次渲染中存放natures，以及以下10行用于去重natureType并存到natureTypes
      this.dependenciesData.nodes.forEach(items => {
        if (natureTypeTemp.indexOf(items.nature) === -1) {
          natureTypeTemp.push(items.nature);
        }
      });
      natureTypeTemp.forEach(items => {
        let singleNature = {
          value: items,
          label: items
        };
        this.natureTypes.push(singleNature);
      });
      // 以下5行用于将获取到的数据的nature按顺序存到natures里
      this.dependenciesData.nodes.forEach(item => {
        this.words.push(item.word);
        let singleNature = { value: item.nature };
        this.natures.push(singleNature);
      });
      this.dependenciesData.relations.forEach(item => {
        this.edges.push(item);
      });
      console.log("finish dispose data");
      this.renderData();
    },
    // 渲染数据
    renderData() {
      console.log("start render data");
      // 先清空原G6画布数据
      document.getElementById("canvasContainer").innerHTML = "";

      // 随机预置颜色
      let prefabColors = [
        { id: 0, word: "#188CFF", stroke: "#91D5FF", fill: "#E6F7FF" }, // 蓝色
        { id: 1, word: "#13C2C2", stroke: "#87E8DE", fill: "#E6FFFB" }, // 薄荷绿
        { id: 2, word: "#52C41A", stroke: "#B7EB8F", fill: "#F6FFED" }, // 柠檬绿
        { id: 3, word: "#722ED1", stroke: "#D3ADF7", fill: "#F9F0FF" }, // 紫色
        { id: 4, word: "#FA541C", stroke: "#FFBB96", fill: "#FFF2E8" }, // 红色
        { id: 5, word: "#FFA113", stroke: "#FAD337", fill: "#FFF5CB" }, // 橙色
        { id: 6, word: "#8199F5", stroke: "#97A9EE", fill: "#EBEFFF" }, // 蓝紫色
        { id: 7, word: "#EB2F96", stroke: "#FFADD2", fill: "#FFF0F6" }, // 桃红色
        { id: 8, word: "#CE7B55", stroke: "#BF9E8F", fill: "#FFEBE2" } // 中褐色
        // { id: 10, word: "orange", stroke: "orange", fill: "orange" }
      ];
      // 词项的统一样式配置
      let stylesOfWordItems = {
        //词项里的文字的配置，如“福建”、“隧道”
        labelCfg: {
          style: {
            fontSize: 18 //词项的文字的大小
          }
        },
        //词项的底色方块大小
        size: [48, 24],
        //词项的方块的配置
        style: {
          fill: "#A5EBFF", //淡青色
          stroke: "#A5EBFF" //淡青色
        },
        anchorPoints: [
          [0, 0], //左上角——index:0
          [1, 0], //右上角——index:1
          [0.5, 1] //下面的点——index:2
        ]
      };
      // 标注的统一样式配置
      let stylesOfTags = {
        //标注里的文字的配置，如“NR”、“NN”
        labelCfg: {
          style: {
            fontSize: 15, //标注的文字的大小,(该G6引擎default 18)
            fill: "000000"
          }
        },
        //标注的底色方块大小
        size: [32, 24], // [32, 24],
        //标注的方块的配置
        style: {
          fill: "#DEE9FF", //极其淡蓝色
          stroke: "#696969", //灰色
          radius: 5
        },
        anchorPoints: [
          [0, 0], //左上角——index:0
          [1, 0], //右上角——index:1
          [0.5, 1], //下面的点——index:2
          [0.5, 0] //下面的点——index:3
        ]
      };
      // 边的统一样式配置
      let stylesOfEdges = {
        sourceAnchor: 3, //1
        targetAnchor: 3, //0
        style: {
          stroke: "#F6BD16", //橙色
          endArrow: {
            path: G6.Arrow.triangle(0, 0, 0),
            d: 0
          },
          startArrow: {
            path: G6.Arrow.triangle(8, 12, 0),
            d: 0,
            fill: "#F6BD16"
          }
        }
      };
      // 括号的统一样式配置
      let stylesOfBrackets = {
        type: "cubic-vertical",
        curvePosition: 0.9,
        style: {
          lineWidth: 1.6,
          stroke: "#adadad",
          startArrow: {
            path: G6.Arrow.triangle(0, 0, 0),
            d: 3
          },
          endArrow: {
            path: G6.Arrow.triangle(0, 0, 0),
            d: 3
          }
        }
      };

      // G6引擎的点和边数据
      /*
       *详见外面我写的《词性标注官网的格式自己修改的.js》
       *
        const data = {
          nodes: [{……},{……}],
          edges: [{……},{……}]
        };
       *
      */

      // 获取G6画布应该展示的大小
      const width = document.getElementById("canvasContainer").scrollWidth;
      const height =
        document.getElementById("canvasContainer").scrollHeight || 280;

      // 边的箭头设置

      // 配置G6引擎
      const graph = new G6.Graph({
        container: "canvasContainer",
        width,
        height,
        linkCenter: false,
        // translate the graph to align the canvas's center, support by v3.5.1
        fitCenter: true,
        defaultNode: {
          size: [35, 25],
          type: "rect",
          style: {
            fill: "#DEE9FF", //极其淡蓝色
            stroke: "#5B8FF9" //微深蓝色
          },
          labelCfg: {
            style: {
              fontSize: 18
            }
          }
        },
        defaultEdge: {
          type: "arc",
          style: {
            stroke: "#F6BD16" //橙色
          },
          anchorPoints: [
            [0.5, 1],
            [0, 0],
            [1, 0]
          ],
          labelCfg: {
            autoRotate: true,
            refY: 5
          }
        },
        modes: {
          // 支持的 behavior
          default: ["drag-node", "drag-canvas"]
        }
      });

      // let res = this.dependenciesData;
      // 组装的数据
      let data = {
        nodes: [],
        edges: []
      };

      let assignedColor = []; // 用于记录颜色顺序
      // // 分配颜色
      // this.natures.forEach((item, index) => {
      //   let singleColor = {
      //     nature: item.value,
      //     id: index % prefabColors.length
      //   };
      //   assignedColor.push(singleColor);
      // });

      // console.log(assignedColor);

      // 这个遍历用于组装【标签】
      this.natures.forEach((item, index) => {
        let singleNode = {
          label: item.value,
          id: JSON.stringify(index),
          x: index * 100,
          y: 0,
          description: "实体",
          labelCfg: stylesOfTags.labelCfg,
          size: stylesOfTags.size,
          // style: stylesOfTags.style,
          style: stylesOfTags.style,
          anchorPoints: stylesOfTags.anchorPoints
        };
        // console.log(singleNode);
        // 这个switch用于判断词项的类型从而设置它框框的尺寸
        // console.log(item);
        switch (this.checkWordType(item.value)) {
          case "中文":
            singleNode.size = [
              stylesOfTags.labelCfg.style.fontSize * item.value.length + 12,
              stylesOfTags.labelCfg.style.fontSize + 12
            ];
            // console.log(item.value + " 是中文，宽度是 " + 24 * item.value.length);
            break;
          case "英文":
            singleNode.size = [
              0.5 * stylesOfTags.labelCfg.style.fontSize * item.length,
              stylesOfTags.labelCfg.style.fontSize
            ];
            // console.log(item.value + " 是英文，宽度是 " + 12 * item.value.length);
            break;
          case "数字":
            singleNode.size = [
              (0.5 * stylesOfTags.labelCfg.style.fontSize + 2) * item.length,
              stylesOfTags.labelCfg.style.fontSize
            ];
            // console.log(item.value + " 是数字，宽度是 " + 14 * item.value.length);
            break;
          case "符号":
            singleNode.size = [
              stylesOfTags.labelCfg.style.fontSize * item.length,
              stylesOfTags.labelCfg.style.fontSize
            ];
            // console.log(item.value + " 是符号，宽度是 " + 24 * item.value.length);
            break;
        }

        // console.log(assignedColor);
        // assignedColor.forEach(singleColor => {
        //   // console.log(singleColor.nature + "  " + item.value);
        //   if (singleColor.nature === item.value) {
        //     // console.log(prefabColors[singleColor.id].fill);
        //     console.log(
        //       "现在是 " +
        //         singleColor.nature +
        //         "  " +
        //         item.value +
        //         "  " +
        //         prefabColors[singleColor.id].fill
        //     );
        //     singleNode.style.fill = prefabColors[singleColor.id].fill;
        //     singleNode.labelCfg.style.fill = prefabColors[singleColor.id].word;
        //     singleNode.style.stroke = prefabColors[singleColor.id].stroke;
        //   } else {
        //     console.log("no");
        //   }
        // });

        // const temp = assignedColor.find(
        //   singleColor => singleColor.nature === item.value
        // );
        // const style = prefabColors[temp.id];
        // Object.assign(singleNode, {
        //   style: {
        //     fill: style.fill,
        //     stroke: style.stroke,
        //     radius: 5
        //   },
        //   labelCfg: {
        //     style: {
        //       fontSize: 15,
        //       fill: style.word
        //     }
        //   }
        // });

        data.nodes.push(singleNode);
      });
      //这个遍历用于组装【词项】
      this.words.forEach((item, index) => {
        let singleNode = {
          label: item,
          id: JSON.stringify(index + this.words.length),
          x: index * 100,
          y: 42,
          description: "标签",
          style: stylesOfWordItems.style,
          size: [48, 24],
          labelCfg: stylesOfWordItems.labelCfg,
          anchorPoints: stylesOfWordItems.anchorPoints
        };
        // 这个switch用于判断词项的类型从而设置它框框的尺寸
        switch (this.checkWordType(item)) {
          case "中文":
            // console.log("中文中文");
            singleNode.size = [
              stylesOfWordItems.labelCfg.style.fontSize * item.length,
              stylesOfWordItems.labelCfg.style.fontSize
            ];
            // console.log(items.word + " 是中文，宽度是 " + 24 * items.word.length);
            break;
          case "英文":
            // console.log(items.word.length);
            singleNode.size = [
              0.5 * stylesOfWordItems.labelCfg.style.fontSize * item.length,
              stylesOfWordItems.labelCfg.style.fontSize
            ];
            // console.log(items.word + " 是英文，宽度是 " + 12 * items.word.length);
            break;
          case "数字":
            // console.log(items.word.length);
            singleNode.size = [
              (0.5 * stylesOfWordItems.labelCfg.style.fontSize + 2) *
                item.length,
              stylesOfWordItems.labelCfg.style.fontSize
            ];
            // console.log(items.word + " 是数字，宽度是 " + 14 * items.word.length);
            break;
          case "符号":
            // console.log(items.word.length);
            singleNode.size = [
              stylesOfWordItems.labelCfg.style.fontSize * item.length,
              stylesOfWordItems.labelCfg.style.fontSize
            ];
            // console.log(items.word + " 是符号，宽度是 " + 24 * items.word.length);
            break;
        }

        data.nodes.push(singleNode);
      });
      //这个遍历用于组装【关系】
      this.edges.forEach((items, index) => {
        let singleEdge = {
          id: "edge" + index,
          source: JSON.stringify(items.start),
          target: JSON.stringify(items.end),
          sourceAnchor: stylesOfEdges.targetAnchor,
          targetAnchor: stylesOfEdges.sourceAnchor,
          label: items.label,
          style: stylesOfEdges.style,
          curveOffset: 0
        };
        //用于判断线的跨度（跨过了段）来决定上凸的量
        {
          switch (Math.abs(parseInt(items.start) - parseInt(items.end))) {
            case 1:
              singleEdge.curveOffset = -15;
              break;
            case 2:
              singleEdge.curveOffset = -40;
              break;
            case 3:
              singleEdge.curveOffset = -65;
              break;
            case 4:
              singleEdge.curveOffset = -90;
              break;
            case 5:
              singleEdge.curveOffset = -110;
              break;
            case 6:
              singleEdge.curveOffset = -135;
              break;
            case 7:
              singleEdge.curveOffset = -150;
              break;
            default:
              singleEdge.curveOffset = -160;
              break;
          }
        }
        //这个if else 用于将那些逆向的线（它是下凹的）转成上凸。（通过修改curveOffset）
        {
          if (parseInt(items.start) - parseInt(items.end) < 0) {
            // console.log("该线条是逆向的");
            singleEdge.curveOffset = -singleEdge.curveOffset;
          } else {
            // console.log("该线条是正向的");
          }
        }
        data.edges.push(singleEdge);
      });
      //这个循环用于组装【括号】
      {
        // console.log(res.nodes.length);
        for (let i = 0; i < this.words.length; i++) {
          let singleBracketLeft = {
            id: "edge" + JSON.stringify(this.edges.length + i * 2 + 0),
            source: JSON.stringify(i),
            target: JSON.stringify(i + this.words.length),
            sourceAnchor: 2,
            targetAnchor: 0,
            type: stylesOfBrackets.type,
            curvePosition: stylesOfBrackets.curvePosition,
            style: stylesOfBrackets.style
          };
          let singleBracketRight = {
            id: "edge" + JSON.stringify(this.edges.length + i * 2 + 1),
            source: JSON.stringify(i),
            target: JSON.stringify(i + this.words.length),
            sourceAnchor: 2,
            targetAnchor: 1,
            type: stylesOfBrackets.type,
            curvePosition: stylesOfBrackets.curvePosition,
            style: stylesOfBrackets.style
          };
          data.edges.push(singleBracketLeft);
          data.edges.push(singleBracketRight);
        }
      }

      // 这个遍历用于美化颜色
      // eslint-disable-next-line no-constant-condition
      if (false) {
        // 根据所分配的颜色列表assignedColor赋予颜色
        data.nodes.forEach(node => {
          // console.log("现在在处理" + node.label);
          assignedColor.forEach(singleColor => {
            if (node.label === singleColor.nature) {
              // 赋予颜色
              // console.log("赋予颜色");
              // console.log(prefabColors);
              // console.log(
              //   node.label +
              //     " 字体颜色 " +
              //     prefabColors[singleColor.id].word +
              //     " 底色 " +
              //     prefabColors[singleColor.id].fill +
              //     " 边框颜色 " +
              //     prefabColors[singleColor.id].stroke
              // );
              node.labelCfg.style.fill = JSON.stringify(
                prefabColors[singleColor.id].word
              );
              node.style.stroke = prefabColors[singleColor.id].stroke;
              node.style.fill = prefabColors[singleColor.id].fill;
            }
          });
        });
      }
      // console.log(data.nodes);

      //G6引擎数据装载
      graph.data(data);
      //G6引擎数据渲染
      graph.render();

      console.log("finish render data");
      // console.log(graph.getEdges());
    },
    // 渲染G6句法依存可视化面板的时候，检测词项里的词的类型（文字、数字、符号、英文）
    checkWordType(word) {
      let patternCHN = new RegExp("[\u4E00-\u9FA5]+");
      let patternENG = new RegExp("[A-Za-z]+");
      var patternNUM = new RegExp("[0-9]+");
      if (patternCHN.test(word)) {
        // console.log(word + "中文");
        return "中文";
      } else if (patternENG.test(word)) {
        // console.log(word + "英文");
        return "英文";
      } else if (patternNUM.test(word)) {
        // console.log(word + "数字");
        return "数字";
      } else {
        // console.log(word + "符号");
        return "符号";
      }
    }
  }
}
</script>

<style>

body {
  background-color: #e1e1e1;
  padding: 10px;
}
.main {
  width: 100%;
  height: 100%;
  background: white;
  padding-top: 100px;
}
#canvasContainer {
  margin: auto;
  width: 80%;
  height: 400px;
  box-shadow: 0px 0px 20px 0px #c1c1c1;
  border-radius: 30px;
}
.buttons {
  border: 1px solid #8a98a130;
  margin-top: 100px;
  height: 100px;
  width: 100%;
  background-color: #87CEFA30;
}
.button {
  margin: auto;
  margin-top: 30px;
  width: 80px;
  height: 40px;
  background: #18f;
  border-radius: 10px;
  cursor: pointer;
  text-align: center;
  color: white;
  font-size: 18px;
  line-height: 36px;
  user-select: none;
}
</style>
