<template>
  <div class="box">
    <!-- 内容 -->
    <div class="content" :class="Preview?'yellow':''" >
      <input type="file" accept=".svg" @change="handleFileUpload" ref="fileInput" style="display: none" />
      <!-- top -->
      <div class="top">
        <img src="./statc/xiugai.png" alt="一键修改" title="一键修改" @click="basics()" class="basicsXiu" />
        <button @click="triggerFileUpload()">导入</button>
        <button @click="copeSvg()">导出</button>
        <button @click="PreviousStep()">撤回上一步</button>
      </div>
      <!-- left -->
      <div class="left">
        <div class="jianch" @click="jiancha()">12</div>
        <div class="yulan" @click="yulan()">{{ Preview?'退出预览':'预览模式'}}</div>
        <div v-if="Preview" class="div32">
          <div @click="hideSwitch()">隐藏所有开关{{ hideSwitchShow }}</div>
          <div @click="hideSwitchOther()">隐藏开关之外的{{ hideSwitchOtherShow }}</div>
          <div @click="kaiguan" v-if="kaiguanShowD">开关{{ kaiguanShow }}</div>
          <div @click="jididaozha" v-if="jididaozhaShowD">接地刀闸{{ jididaozhaShow }}</div>
          <div @click="daozha" v-if="daozhaShowD">刀闸{{ daozhaShow }}</div>
        </div>
        <!-- <div v-for="(v, i) in 9" :key="i" class="div12" @click="Modify(i)"></div> -->
      </div>
      <!-- right -->
      <div class="right"></div>
      <!-- bottom -->
      <div class="bottom">
        <div class="flex" v-if="!Preview">
          <!-- <div>组件：</div> -->
          <div v-for="(item, index) in toolColor" :key="index" :style="'background-color:' + item.color" class="div12" @click="cclier(item)">
            <!-- {{ item.name.match(/\d+/g)[0] }} -->
            {{ item.name.split('kv')[1] }}
          </div>
          <!-- 右边， 点击的组件的所有类型 -->
          <div  class="useCom">
            <div v-for="(v,i) in componentType" :key="v" :class="i===componentIndex?'selecCom':''" @click="ModifyComponent(i)">{{ i+1 }}{{ componentIndex }}</div>
          </div>
          <div class="ScaleRatioColumn">{{ Math.round(scale * 100) }}%</div>
        </div>
      </div>
      <!-- 内容 -->
      <div class="conentBox" id="wrapper" ref="wrapper">
        <!-- https://github.com/chuxiaoguo/vue-sketch-ruler/blob/master/docs/src/components/UserRuler.vue -->
        <SketchRule
          :thick="thick"
          :scale="scale"
          :width="width"
          :height="height"
          :startX="startX"
          :startY="startY"
          :shadow="shadow"
          :horLineArr="lines.h"
          :verLineArr="lines.v"
          :cornerActive="true"
          :isShowRuler="false"
          :isShowReferLine="false"
          :palette="palette"
        >
        </SketchRule>
        <div ref="screensRef" id="screens" @wheel="handleWheel" @scroll="handleScroll">
          <div
            ref="containerRef"
            class="screen-container"
            :style="{
              width: svgScaleWidth + 'px',
              height: svgScaleHeight + 'px'
            }"
            @click="svgClick"
            @mousedown="startSelection"
            @mouseup="updateSelection"
            @mouseover="mouseover"
            @mouseout="mouseout"
            @contextmenu.prevent="handleRightClick"
          >
            <div class="selection-box" :style="selecFrame.selectionStyle" v-if="selecFrame.isSelecting">
              <div class="topPO">{{ Math.trunc(selecFrame.selectionStyle.left.split("px")[0]) }}x{{ Math.trunc(selecFrame.selectionStyle.top.split("px")[0]) }}</div>
              <div class="bottPO">{{ Math.trunc(+selecFrame.selectionStyle.left.split("px")[0] + +selecFrame.selectionStyle.width.split("px")[0]) }}x{{ Math.trunc(+selecFrame.selectionStyle.top.split("px")[0] + +selecFrame.selectionStyle.height.split("px")[0]) }}</div>
            </div>
            <div v-html="newSvgContent" id="canvas" :style="canvasStyle"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SketchRule from "vue-sketch-ruler";
export default {
  data() {
    return {
      palette:{
            bgColor: 'rgba(225,225,255, 0)',
            longfgColor: '#bbb',
            shortfgColor: '#bbb',
            fontColor: '#bbb',
            shadowColor: 'red',
            lineColor: '#ccc',
            borderColor: '#00000000',
            cornerActiveColor: 'red'
      },
      newSvgContentCopy:{},
      Preview:false,
      originalText: "svgManualEncryption",
      EncryptContent: ["嚎缧凧博禇盩峮隧揋窜诐奤皣覿粋纫", "厫畃窾乶頔", "儺嬡爫絯彴嚐", "仿歗缶纑纲柪", "仗皂洦粶纾", "收雪弲沯", "仸亀剰蠥", "昉肋耳勥", "坛绉皶見", "焔晸揀剻", "勛珙皶揪", "寺儞陕匦", "添附皶見", "卉垩畏畸赾茙", "寭阳杮勒恚勎", "偏玛呪匘", "隵揑駱頸", "词奱皶揪粚纱", "隵揑窾",'剅嚈仝','寒桎仝'],
      hoveredIndex: null, // 跟踪悬停组件的索引
      newSvgContent: null,
      Svgname: "",
      embedLoading: false, // 加载
      SVGTitle: "", // 窗口标题
      currentSvg: null, // SVG地址
      oldTargetNode: [], // 记录点击的节点
      jumpSVGData: {}, // 记录跳转的数据
      // 新加的选中区域
      startX: 0,
      startY: 0,
      endX: 0,
      endY: 0,
      selectedLines: [], // 存储选中线条的引用或ID
      threeModel: [],
      // 选中框的实现
      selecFrame: {
        isSelecting: false,
        selectionStyle: {
          position: "absolute",
          left: "0px",
          top: "0px",
          width: "0px",
          height: "0px",
          backgroundColor: "rgba(255, 0, 0, 0.3)",
          border: "1px solid red"
        }
      },
      // 尺子
      thick: 18,
      scale: 1,
      width: 1820,
      height: 849, //尺子div的大小
      startX: 0, //刻度的xy轴
      startY: 0,
      // 刻度的标线，不需要
      lines: {
        h: [],
        v: []
      },
      // 结束
      presetLine: [
        { type: "l", site: 100 },
        { type: "v", site: 200 }
      ],
      svgDoc: "",
      toolColor: [
        {
          color: "rgb(0,0,255)",
          name: "kv1000kV"
        },
        {
          color: "rgb(128,0,128)",
          name: 'kv220kV'
        },
        {
          color: "rgb(240,65,85)",
          name: 'kv110kV'
        },
        {
          color: "rgb(255,255,0)",
          name: 'kv35kV'
        },
        {
          color: "rgb(185,72,66)",
          name: 'kv10kV'
        },
        {
          color: "rgb(0,0,139)",
          name: 'kv6kV'
        },
        {
          color: "rgb(128,128,128)",
          name: 'kv0kV'
        }
      ],
      svgWidth: 0,
      svgHeight: 0,
      svgScaleHeight: 0,
      svgScaleWidth: 0,
      // svg回退
      svgRecordStack: [],
      modifyEvent: [],
      hideSwitchShow: false,
      hideSwitchOtherShow:false,
      // 存储所有的svg组件的id
      svgId: [],
      // 组件的状态
      kaiguanShow: false,
      daozhaShow: false,
      jididaozhaShow: false,
      kaiguanShowD:false,
      daozhaShowD:false,
      jididaozhaShowD: false,
      symbol: [],
      componentType: [],
      componentIndex:'',
    };
  },
  components: {
    SketchRule
  },
  computed: {
    shadow() {
      return {
        x: 0,
        y: 0,
        width: 0,
        height: 0
      };
    },
    canvasStyle() {
      return {
        width: this.svgWidth + "px",
        height: this.svgHeight + "px",
        transform: `scale(${this.scale})`
      };
    }
  },
  methods: {
    handleUseClick (event) {
      console.log('ffffffff',event)
    },
    kaiguan () {
      this.kaiguanShow = !this.kaiguanShow
      const LayerG = $('#canvas #DollyBreaker_Layer>g>use')
      LayerG.each((i,v) => {
        const oldxhref = v.getAttribute("xlink:href")
        let newxhref=""
        if (this.kaiguanShow) {
          if (oldxhref.indexOf('0_0') != -1) {
              newxhref = oldxhref.substring(0, oldxhref.length - 1) + '2';
            } else {
              newxhref = oldxhref.substring(0, oldxhref.length - 1) + '1';
            }
        } else {
          	if (oldxhref.indexOf('0_2') != -1) {
              newxhref = oldxhref.substring(0, oldxhref.length - 1) + '0';
            } else {
              newxhref = oldxhref.substring(0, oldxhref.length - 1) + '0';
            }
        }
        v.setAttribute("xlink:href", newxhref )
        // 正常只有这几种状态，其他状态没有做处理
          if (!['1_0_0', '1_0_2', '1_1_0', '1_1_1'].includes(oldxhref.slice(-5))) {
            alert('竟然有其他开关状态：' + oldxhref)
          }
      })
      
      const BreakerLayer = $('#canvas #Breaker_Layer>g>use')
      BreakerLayer.each((i,v) => {
        let oldxhref = v.getAttribute("xlink:href")
        let newxhref = ""
        // 正常只有这几种状态，其他状态没有做处理
        if (!['_0', '_1'].includes(oldxhref.slice(-2))) {
          alert('竟然有其他开关状态：' + oldxhref)
        }
        if (this.kaiguanShow) {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1)
          newxhref = oldxhref + '1';
        } else {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1)
          newxhref = oldxhref + '0';
        }
        v.setAttribute("xlink:href", newxhref )
      })
    },
    jididaozha () {
      this.jididaozhaShow = !this.jididaozhaShow
      const GroundDisconnectorG = $('#canvas #GroundDisconnector_Layer>g>use')
      GroundDisconnectorG.each((i,v) => {
        let oldxhref = v.getAttribute("xlink:href")
        let newxhref = ""
        // 正常只有这几种状态，其他状态没有做处理
        if (!['_0', '_1'].includes(oldxhref.slice(-2))) {
          alert('竟然有其他接地刀闸状态：' + oldxhref)
        }
        if (this.jididaozhaShow) {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1)
          newxhref = oldxhref + '1';
        } else {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1)
          newxhref = oldxhref + '0';
        }
        v.setAttribute("xlink:href", newxhref )
      })
    },
    daozha () {
      this.daozhaShow = !this.daozhaShow
      const DisconnectorG = $('#canvas #Disconnector_Layer>g>use')
      DisconnectorG.each((i,v) => {
        let oldxhref = v.getAttribute("xlink:href")
        let newxhref = ""
        // 正常只有这几种状态，其他状态没有做处理
        if (!['_0', '_1'].includes(oldxhref.slice(-2))) {
          alert('竟然有其他刀闸状态：' + oldxhref)
        }
        if (this.daozhaShow) {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1)
          newxhref = oldxhref + '1';
        } else {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1)
          newxhref = oldxhref + '0';
        }
        v.setAttribute("xlink:href", newxhref )
      })
    },
    // 隐藏所有开关
    hideSwitch () {
      this.hideSwitchShow = !this.hideSwitchShow
      if (this.hideSwitchShow) {
          $('#DollyBreaker_Layer').hide()
          $('#GroundDisconnector_Layer').hide()
          $('#Disconnector_Layer').hide()
          $('#Breaker_Layer').hide()
        } else {
          $('#DollyBreaker_Layer').show()
          $('#GroundDisconnector_Layer').show()
          $('#Disconnector_Layer').show()
          $('#Breaker_Layer').show()
        }
    },
    // 隐藏开关之外的
    hideSwitchOther () {
      this.hideSwitchOtherShow = !this.hideSwitchOtherShow
     		if (this.hideSwitchOtherShow) {
			this.svgId.forEach(v => {
				if (!['Head_Layer', 'DollyBreaker_Layer', 'GroundDisconnector_Layer', 'Disconnector_Layer','Breaker_Layer'].includes(v)) {
					$(`#${v}`).hide()
				}
			})
		} else {
			this.svgId.forEach(v => {
				if (!['Head_Layer', 'DollyBreaker_Layer', 'GroundDisconnector_Layer', 'Disconnector_Layer','Breaker_Layer'].includes(v)) {
					$(`#${v}`).show()
				}
			})
		}

    },
    yulan () {
      this.Preview = !this.Preview
      if (this.Preview) {
        console.log('备份')
        // 将svg数据备份
        this.newSvgContentCopy = $('#canvas>svg').clone(true)[0];
        this.$refs.containerRef.style.pointerEvents = "none"
        // 关闭框选弹窗
        this.handleRightClick()
      } else {
        // svg还原
        console.log($('#canvas>svg')[0])
        $('#canvas>svg').html(this.newSvgContentCopy)
        
        this.$refs.containerRef.style.pointerEvents="all"
      }

    },
    // 动作返回上一步
    PreviousStep() {
      console.log(this.svgRecordStack);
      return
      this.newSvgContent = this.svgRecordStack.pop();
    },
    // 导出svg
    copeSvg() {
      console.log(this.newSvgContent, "this.newSvgContent");
      if (!this.newSvgContent) {
        alert("请先导入svg文件");
        return;
      }
      // 创建一个Blob对象
      const blob = new Blob([this.newSvgContent], {
        type: "image/svg+xml;charset=utf-8"
      });

      // 创建一个数据URL
      const url = URL.createObjectURL(blob);

      // 创建一个链接元素
      const link = document.createElement("a");
      link.href = url;
      link.download = this.Svgname; // 设置下载的文件名

      // 触发下载
      document.body.appendChild(link);
      link.click();

      // 清理
      URL.revokeObjectURL(url);
      document.body.removeChild(link);
    },
    // 鼠标移入
    mouseover(event) {
      return;
      // 检查事件目标是否有 initshowingplane 属性
      if (event.target.tagName.match(/^(rect|circle|path|use)$/) && !event.target.hasAttribute("initshowingplane")) {
        // 执行你的操作，比如显示边框
        // console.log('鼠标移入', event.target)
        event.target.classList.add("bordered");
        event.target.setAttribute("stroke", "black"); // 设置边框颜色
        event.target.setAttribute("stroke-width", "2"); // 设置边框宽度
        const index = event.target.getAttribute("data-index");
        this.hoveredIndex = index; // 记录悬停的组件索引
        // ...
      }
    },
    // 鼠标移出
    mouseout(event) {
      return;
      if (event.target.tagName.match(/^(rect|circle|path|use)$/) && !event.target.hasAttribute("initshowingplane")) {
        event.target.classList.remove("bordered");
        this.hoveredIndex = null; // 取消悬停
        // event.target.removeAttribute('stroke'); // 或者设置回原来的stroke颜色和宽度
      }
      // console.log('鼠标移出',event)
    },
    // 按下鼠标
    svgClick(event) {
      
        // 如果是拖拽，我就清除底部的组件id切换
        this.componentType= []
      this.componentIndex = ''
        
      if (!this.selecFrame.isSelecting) {
        // 点击我就清除之前的所有数据
        this.handleRightClick()
      }
      // 如果是点击的组件
      if (Math.trunc(this.selecFrame.selectionStyle.left.split("px")[0])===Math.trunc(+this.selecFrame.selectionStyle.left.split("px")[0] + +this.selecFrame.selectionStyle.width.split("px")[0]) && Math.trunc(this.selecFrame.selectionStyle.top.split("px")[0])===Math.trunc(+this.selecFrame.selectionStyle.top.split("px")[0] + +this.selecFrame.selectionStyle.height.split("px")[0])) {
        console.log('点击的同一个位置', event.target)
        if (event.target.tagName === 'use') {
          const href = event.target.getAttribute('xlink:href')
          const hrefSlice=href.slice(1,-1)
          const arrUs = []
          this.symbol.forEach(v => {
            if (v.includes(hrefSlice)) {
              arrUs.push(v)
            }
          })
          this.componentType = arrUs
          this.componentIndex = arrUs.findIndex(v => v === href.slice(1))
          console.log(href)
          console.log(arrUs)
          console.log(this.componentIndex)
        }
        this.newSelectedModel = [event.target]
        // 显示切换组件类型
        // this.newSelectedModel=[event.target]
      }
    },
    // 按下鼠标左键：
    startSelection(event) {
      console.log("12445按下：", event.clientX, this.$refs.containerRef.getBoundingClientRect().left);

      this.startX = event.clientX - this.$refs.containerRef.getBoundingClientRect().left;
      this.startY = event.clientY - this.$refs.containerRef.getBoundingClientRect().top;
      console.log("按下坐标：", this.startX, this.startY);
      this.selecFrame.isSelecting = false;
      // 绘制选中框
      window.addEventListener("mousemove", this.updateSelecmove);
    },
    // 鼠标右键
    handleRightClick() {
      console.log("鼠标右键");
      this.selecFrame.isSelecting = false;
      this.startX = 0;
      this.startY = 0;
      this.end = 0;
      this.endY = 0;
      this.newSelectedLines = [];
      // 选中的组件
      this.newSelectedModel = [];
      this.selecFrame = {
        isSelecting: false,
        selectionStyle: {
          position: "absolute",
          left: "0px",
          top: "0px",
          width: "0px",
          height: "0px",
          backgroundColor: "rgba(255, 0, 0, 0.3)",
          border: "1px solid red"
        }
      };
    },
    // 鼠标移动，绘制选中框
    updateSelecmove(event) {
      this.selecFrame.isSelecting = true;
      this.endX = event.clientX - this.$refs.containerRef.getBoundingClientRect().left;
      this.endY = event.clientY - this.$refs.containerRef.getBoundingClientRect().top;
      // 根据startX, startY, endX, endY确定选中的线条，并更新selectedLines
      const left = Math.min(this.startX, this.endX);
      const top = Math.min(this.startY, this.endY);
      const Mleft = Math.max(this.startX, this.endX);
      const Mtop = Math.max(this.startY, this.endY);

      const width = Math.abs(this.endX - this.startX);
      const height = Math.abs(this.endY - this.startY);
      this.selecFrame.selectionStyle.left = left + "px";
      this.selecFrame.selectionStyle.top = top + "px";

      this.selecFrame.selectionStyle.width = width + "px";
      this.selecFrame.selectionStyle.height = height + "px";
    },
    // 弹起坐标，结束绘制框监听
    updateSelection(event) {
      console.log("弹起");
      const left = Math.min(this.startX, this.endX);
      const top = Math.min(this.startY, this.endY);
      const Mleft = Math.max(this.startX, this.endX);
      const Mtop = Math.max(this.startY, this.endY);
      // console.log("对比：", left, top, Mleft, Mtop);
      this.newSelectedLines = this.getIntersectingLines(left, top, Mleft, Mtop);
      this.newSelectedModel = this.getIntersectingmodel(left, top, Mleft, Mtop);
      // console.log("选中的内容", this.newSelectedLines, this.newSelectedModel);
      window.removeEventListener("mousemove", this.updateSelecmove);
    },
    // 修改组件的id。 
    ModifyComponent(v) {
      if (this.newSelectedModel.length === 1) {
        console.log(this.newSelectedModel[0])
        console.log(this.componentType[v])
        this.newSelectedModel[0].setAttribute('xlink:href', '#' + this.componentType[v])
        this.componentIndex=v
      }
    },
    // 返回选中的组件
    getIntersectingmodel(startX, startY, endX, endY) {
      const intersectin = [];
      // 1. 拿到所有的线条
      const Model = this.svgDoc.querySelectorAll("use");
      // console.log('线条',lines)
      // 定义一个函数来检查点是否在矩形内
      function isPointInRect(x, y) {
        // console.log(x,y,startX,endX,startY,endY)
        return x >= startX && x <= endX && y >= startY && y <= endY;
      }
      // 定义一个函数来检查线段是否与矩形相交
      const doesLineIntersectRect = (v) => {
        // 获取线段的两个端点坐标
        const sx = parseInt(v.getAttribute("x")) * this.scale; // 获取 d 属性
        const sy = parseInt(v.getAttribute("y")) * this.scale; // 获取 d 属性
        const ex = sx + parseInt(v.getAttribute("width")) * this.scale; // 获取 d 属性
        const ey = sy + parseInt(v.getAttribute("height")) * this.scale; // 获取 d 属性

        // 检查线段的两个端点是否在矩形内
        if (isPointInRect(sx, sy) || isPointInRect(ex, ey)) {
          return true; // 如果一个端点在矩形内，则线段与矩形相交
        }
        return false; // 如果没有找到相交情况，则返回 false
      };
      for (const mode of Model) {
        if (doesLineIntersectRect(mode)) {
          intersectin.push(mode);
        }
      }
      return intersectin;
    },
    // 返回选中区域的线条
    getIntersectingLines(startX, startY, endX, endY) {
      // console.log("选中的线条", startX, startY, endX, endY);
      const intersectingLines = [];
      // 1. 拿到所有的线条
      const lines = this.svgDoc.querySelectorAll("path");
      // 定义一个函数来检查点是否在矩形内(可以不用全在框中，只要有一半区域在就行)
      function isPointInRect(x, y) {
        return x >= startX && x <= endX && y >= startY && y <= endY;
      }
      // 返回线条的起点和终点坐标
      const parsePath = (pathString) => {
        const commands = pathString.match(/[ML]/g);
        const parameters = pathString.match(/-?\d+(?:\.\d+)?/g).map(Number);
        // console.log('选中组件',parameters)
        const lineSegments = {};
        let currentX = 0,
          currentY = 0; // 初始坐标

        for (let i = 0; i < commands.length; i++) {
          const cmd = commands[i];
          let nextX, nextY;

          if (cmd === "M") {
            // moveto 命令设置新的当前点
            if (i === 0 || (parameters[i * 2 - 2] !== currentX && parameters[i * 2 - 1] !== currentY)) {
              // 只有在坐标变化时才添加线段（忽略连续的相同坐标）
              // lineSegments.push({
              //   startX: currentX,
              //   startY: currentY,
              //   endX: parameters[i * 2],
              //   endY: parameters[i * 2 + 1],
              // });
              (lineSegments["startX"] = parameters[i * 2]), (lineSegments["startY"] = parameters[i * 2 + 1]);
            }
            currentX = parameters[i * 2] * this.scale;
            currentY = parameters[i * 2 + 1] * this.scale;
          } else if (cmd === "L") {
            // console.log("this", this);
            // lineto 命令从当前点到新点绘制线段
            nextX = parameters[i * 2] + (cmd === "l" ? currentX : 0);
            nextY = parameters[i * 2 + 1] + (cmd === "l" ? currentY : 0);
            lineSegments["endX"] = nextX * this.scale;
            lineSegments["endY"] = nextY * this.scale;
            currentX = nextX;
            currentY = nextY;
          }
        }
        return lineSegments;
      };

      // 定义一个函数来检查线段是否与矩形相交
      function doesLineIntersectRect(line) {
        // 获取线段的两个端点坐标
        const pathData = line.getAttribute("d"); // 获取 d 属性
        const points = parsePath(pathData); // 解析路径并获取点
        // 检查线段的两个端点是否在矩形内
        if (isPointInRect(points.startX, points.startY) || isPointInRect(points.endX, points.endY)) {
          return true; // 如果一个端点在矩形内，则线段与矩形相交
        }
        return false; // 如果没有找到相交情况，则返回 false
      }
      // console.log("所有的线条", lines);
      // 2. 遍历所有线条并检查是否相交
      for (const line of lines) {
        if (doesLineIntersectRect(line)) {
          intersectingLines.push(line);
        }
      }

      return intersectingLines;
    },
    // 获取div的大小
    initSize() {
      const wrapperRect = document.querySelector("#wrapper").getBoundingClientRect();
      const borderWidth = 1;
      this.width = wrapperRect.width - this.thick - borderWidth;
      this.height = wrapperRect.height - this.thick - borderWidth;
    },
    // 控制缩放值
    handleWheel(e) {
      if (e.ctrlKey || e.metaKey) {
        e.preventDefault();
        // 初步计算新的缩放比例
        let newScale = this.scale - e.deltaY / 500;
        // 确保缩放比例不会低于 0.2
        newScale = Math.max(0.2, newScale);
        // 如果需要，确保缩放比例也不会超过 2（这一步在您的情况下可能是可选的，因为从 0.2 减去任何正值都不会使结果超过 2）
        // 但为了清晰和完整性，我还是包含了它
        newScale = Math.min(2, newScale);
        // 现在将 newScale 转换为具有两位小数的浮点数（如果需要的话）
        const nextScale = parseFloat(newScale.toFixed(2));
        this.scale = nextScale;
        this.svgScaleWidth = this.svgWidth * this.scale;
        this.svgScaleHeight = this.svgHeight * this.scale;
      }
      this.$nextTick(() => {
        this.handleScroll();
      });
    },
    handleScroll() {
      const screensRect = document.querySelector("#screens").getBoundingClientRect();
      const canvasRect = document.querySelector("#canvas").getBoundingClientRect();

      // 标尺开始的刻度
      const startX = (screensRect.left + this.thick - canvasRect.left) / this.scale;
      const startY = (screensRect.top + this.thick - canvasRect.top) / this.scale;

      this.startX = startX >> 0;
      this.startY = startY >> 0;
    },
    // 选中颜色 修改组件和线条。修改三色组件
    cclier(v) {
        // 修改线条颜色
        this.newSelectedLines.forEach((lineId) => {
          lineId.setAttribute("stroke", v.color);
        });
        // 修改组件颜色
        this.newSelectedModel.forEach((lineId) => {
          lineId.setAttribute("class", v.name);
          console.log('选中修改颜色',lineId)
        });
        this.selecFrame = this.$options.data().selecFrame;
        this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
        this.newSelectedLines = [];
        this.newSelectedModel = [];
      this.svgRecordStack.push(this.newSvgContent);
    },
    // 文字的加密解密
    xorEncryptDecrypt(text) {
      // 转换key为字符数组，以便可以逐个字符处理
      let keyChars = this.originalText.split("");
      // 加密/解密时使用的key索引
      let keyIndex = 0;
      // 将text转换为字符数组
      let textChars = text.split("");
      // 用于存放加密/解密后的结果
      let result = "";
      // 遍历text的每一个字符
      for (let i = 0; i < textChars.length; i++) {
        // 获取当前字符的Unicode编码
        let charCode = textChars[i].charCodeAt(0);
        // 获取key的当前字符的Unicode编码，如果key不够长，则循环使用
        let keyCharCode = keyChars[keyIndex % keyChars.length].charCodeAt(0);
        // 使用异或进行加密/解密
        let xorResult = charCode ^ keyCharCode;
        // 将加密/解密后的Unicode编码转换为字符
        result += String.fromCharCode(xorResult);
        // 移动到key的下一个字符
        keyIndex++;
      }
      return result;
    },
    // 修改事件，单个修改
    Modify () { },
    jiancha () {
       const useElements = document.querySelectorAll('svg use');
              useElements.forEach((el) => {
                el.setAttribute('pointer-events', 'bounding-box');
              });
              return
      // 开关
      const elements1 = this.svgDoc.querySelectorAll("#DollyBreaker_Layer>g>use");
      // 接地刀闸
      const elements2 = this.svgDoc.querySelectorAll("#GroundDisconnector_Layer>g>use");
      // 刀闸
      const elements3 = this.svgDoc.querySelectorAll("#Disconnector_Layer>g>use");
      let elementss1 =[]
      let elementss2 =[]
      let elementss3 =[]
      elements1.forEach(v => {
        elementss1.push(v.getAttribute("xlink:href"))
      })
      elements2.forEach(v => {
        elementss2.push(v.getAttribute("xlink:href"))
      })
      elements3.forEach(v => {
        elementss3.push(v.getAttribute("xlink:href"))
      })
      console.log('开关',[...new Set(elementss1)])
      console.log('接地刀闸',[...new Set(elementss2)])
      console.log('刀闸',[...new Set(elementss3)])
      
    },
    // 基础修改
    basics() {
      // 1. 删除选中文字
      const elements1 = this.svgDoc.querySelectorAll("#Text_Layer>text");
      // 存储所有匹配到文本的y轴，用于删除没有匹配到的文本，根据当前y轴来判断
      const textY = [];
      this.EncryptContent.forEach((cc) => {
        elements1.forEach((textElement) => {
          // 只要文本中包含该文字
          if (textElement.textContent.trim().includes(cc)) {
            // 如果已经被删除，就跳过
            if (!textElement.parentNode) return;
            // console.log(textElement.textContent.trim());
            textY.push(textElement.getAttribute("y"));
            // 删除当前标签
            textElement.parentNode.removeChild(textElement);
          }
        });
      });
      const contrastResult = this.contrast(textY);
      // 第二次匹配，将删除的坐标轴附近的参数，也删除
      elements1.forEach((cy) => {
        // 如果已经被删除，就跳过
        if (!cy.parentNode) return;
        if (this.contrastArr(cy.getAttribute("y"), contrastResult, 7)) {
          cy.parentNode.removeChild(cy);
        }
      });
      // 2.删除所有图片
      const elements2 = this.svgDoc.querySelectorAll("#Other_Layer>image");
      elements2.forEach((textElement) => {
        // 删除当前标签
        textElement.parentNode.removeChild(textElement);
      });
      // 3. 修改背景颜色
      const elements3 = this.svgDoc.querySelectorAll("#Head_Layer>rect");
      elements3.forEach(function (element) {
        // 设置填充颜色为蓝色
        element.setAttribute("fill", "rgb(0,47,71)");
      });
      // 4.修改遮挡背景
      const elements4copy = this.svgDoc.querySelectorAll("#Link_Layer>g>path");
      const elements4s = this.svgDoc.querySelectorAll("#ACLineSegment_Layer>g>path");
      const elements4 = [...elements4s, ...elements4copy];
      elements4.forEach((v) => {
        // 获取d属性的内容
        const dAttributeContent = v.getAttribute("d");
        // 使用空格切割字符串
        const pathCommands = dAttributeContent.split(" ");
        // 只要长度大于等于9，就代表为一个闭合的div
        if (pathCommands.length >= 9) {
          v.style["fill"] = "none";
        }
      });
      // 5. 删除组件内的颜色
      const elements5 = this.svgDoc.querySelectorAll("defs>symbol>*");
      const Protect_LayerUse = this.svgDoc.querySelectorAll("#Protect_Layer>g>use");
      console.log('组件1',Protect_LayerUse)
      let circleName=[]
      Protect_LayerUse.forEach(v => {
        circleName.push(v.getAttribute("xlink:href"))
      })
      console.log('组件2',Protect_LayerUse)
      circleName = [...new Set(circleName)]
      console.log('组件3',circleName)
      const circleNa = circleName.map(v => {
        v = v.split('#')[1]
        return v
      })
      console.log(circleNa)
      // console.log("删除组件内颜色", elements5);
      if (circleNa.length) {
        circleNa.forEach((ecve) => {
          elements5.forEach((element) => {
            console.log(element.parentNode.getAttribute("id"),ecve)
            // 获取stroke属性的内容
            if (ecve !== element.parentNode.getAttribute("id")) {
              element.setAttribute("stroke", "");
            }
            // 点点点的默认组件颜色还不能删
          });
        })
      } else {
        elements5.forEach((element) => {
              element.setAttribute("stroke", "");
          });
      }

      // 6.删除可以跳转的标签
      const hrefSkip = this.svgDoc.querySelectorAll("a");
      hrefSkip.forEach((v) => {
        v.removeAttribute("xlink:href");
      });

      // 7. 删除最底部的线条
      const bottomLine = this.svgDoc.querySelectorAll("line");
      const backHeight = this.svgDoc.querySelector("#Head_Layer>rect");
      const Backhei = parseInt(backHeight.getAttribute("height"));
      bottomLine.forEach((v) => {
        // console.log('线条',v.getAttribute("y2"))
        if (Backhei - parseInt(v.getAttribute("y2")) < 40) {
          v.parentNode.removeChild(v);
          // console.log('获取到了,可以删除', v)
          // v.setAttribute('stroke', 'red')
          // v.setAttribute('stroke-width', '20');
        }
        // v.removeAttribute("xlink:href");
      });

      // 9. 删除右边的内容。下角的点点点的。并加文本
      const GZPLayer = this.svgDoc.getElementById("GZP_Layer"); // 使用ID直接获取<g>元素
      if (GZPLayer) {
        // 9.1
        const BackW = backHeight.getAttribute("width");
        // 获得文字的左右边距，
        const BackPadd = BackW - 500;
        // 删除数字
        const MeasurementValue_Layer = this.svgDoc.querySelectorAll("#MeasurementValue_Layer text");
        // console.log("删除数字14545", MeasurementValue_Layer);
        MeasurementValue_Layer.forEach((textElement) => {
          if (textElement.getAttribute("x") > BackPadd) {
            // console.log("数字", textElement.textContent.trim());
            // 删除当前标签
            textElement.parentNode.removeChild(textElement);
          }
        });
        // 删除文字
        elements1.forEach((textElement) => {
          if (textElement.getAttribute("x") > BackPadd && !this.EncryptContent.includes(textElement.textContent.trim())) {
            // console.log("文字", textElement, textElement.parentNode);
            // 删除当前标签
            textElement.parentNode.removeChild(textElement);
          }
        });

        // 9.2删除点点点
        GZPLayer.parentNode.removeChild(GZPLayer);
      }
      // 9. 删除值班名单下的背景框
      // 假设svg是SVG根元素的引用，或者您可以直接引用到ID为Other_Layer的<g>元素
      const group = this.svgDoc.getElementById("Other_Layer"); // 使用ID直接获取<g>元素
      // 查找该<g>元素下具有特定afmask属性且没有changepicplane属性的<a>元素
      const aElement = group.querySelector('a[AFMask="39039"]:not([ChangePicPlane])');
      // 如果找到了这样的<a>元素，则查找其下的第一个<rect>元素
      if (aElement) {
        const firstRect = aElement.querySelector("rect");
        if (firstRect) {
          // 现在firstRect是第一个<rect>元素的引用
          console.log(firstRect); // 可以在控制台中查看这个元素
          firstRect.parentNode.removeChild(firstRect);
        }
      } else {
        // 使用第二种方法
        const aElement2 = group.querySelectorAll("a:not([ChangePicPlane])");
        aElement2.forEach(function (va) {
          // 检查fill属性是否为rgb(0,85,0)
          const rect = va.querySelector("rect");
          // 注意：这里我们直接将属性值作为字符串比较，这适用于简单的颜色值
          // 对于更复杂的颜色匹配，可能需要更复杂的解析逻辑
          if (rect.getAttribute("fill") === "rgb(0,85,0)") {
            // 获取x属性的值，并转换为数字进行比较
            if (parseInt(rect.getAttribute("x"), 10) < 100) {
              // 如果满足条件，则可以进行一些操作，比如改变其样式
              console.log("第二次找到的", rect);
              rect.parentNode.removeChild(rect);
            }
          }
        });
      }

      const useElements = this.svgDoc.querySelectorAll('svg use');
            console.log(useElements)
            useElements.forEach((el) => {
              el.setAttribute('pointer-events', 'bounding-box');
              // console.log(el)
              });
      this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
      this.svgRecordStack.push(this.newSvgContent);
    },
    // 数组的差异对比
    contrast(data) {
      // 将数据转换为数值并排序
      const sortedData = data.map(Number).sort((a, b) => a - b);

      // 计算中位数（用于确定“差异过大”的阈值）
      const median = sortedData.length % 2 === 0 ? (sortedData[sortedData.length / 2 - 1] + sortedData[sortedData.length / 2]) / 2 : sortedData[Math.floor(sortedData.length / 2)];

      // 假设“差异过大”的阈值是中位数的一半（这个值可以根据需要调整）
      const threshold = median / 2;

      // 将数据分成两个数组
      let lowerArray = [];
      let upperArray = [];
      for (let num of sortedData) {
        if (Math.abs(num - median) <= threshold) {
          lowerArray.push(num);
        } else {
          upperArray.push(num);
        }
      }

      // 提取每个数组的中间元素
      function getMiddleElement(arr) {
        const midIndex = Math.floor(arr.length / 2);
        return arr.length % 2 === 0
          ? (arr[midIndex - 1] + arr[midIndex]) / 2 // 偶数长度时取中间两个的平均值
          : arr[midIndex]; // 奇数长度时取中间那个
      }
      const arrData = [];
      arrData[0] = getMiddleElement(lowerArray);
      arrData[1] = getMiddleElement(upperArray);
      return arrData;
    },
    // 数据名称对比
    contrastArr(y, arr, offset) {
      // 将数组中的字符串转换为数字
      const numbers = arr.map(Number);
      // 将y也转换为数字
      const target = Number(y);
      // 检查数组中的每个数字是否在范围内
      for (let num of numbers) {
        if (Math.abs(num - target) <= offset) {
          return true; // 如果在范围内，则返回true
        }
      }
      return false; // 如果没有找到匹配项，则返回false
    },
    // 文件input调用
    triggerFileUpload() {
      this.$refs.fileInput.click();
    },
    // 修改名称
    repName(filename) {
      console.log("传入的名称", filename);
      // 移除前缀 "CD." 和后缀 ".fac"
      const nameWithoutPrefixSuffix = filename.replace(/^CD\./, "").replace(/\.fac/, "");
      // 替换下划线为点，并添加新前缀 "简易修改版" 和新后缀 ".svg"
      const newName = `${nameWithoutPrefixSuffix.replace(/_/g, ".")}`;
      return newName;
    },
    // 引入一个svg
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file && file.type === "image/svg+xml") {
        const reader = new FileReader();
        this.Svgname = this.repName(file.name);
        // console.log("获取到svg名称", this.Svgname);
        reader.onload = (e) => {
          this.newSvgContent = e.target.result;
          console.log(e.target.result)
          this.$nextTick(() => {
            const parser = new DOMParser();
            // 解析SVG字符串为DOM对象
            this.svgDoc = parser.parseFromString(this.newSvgContent, "image/svg+xml");
            this.svgRecordStack.push(this.newSvgContent);
            const svg = this.svgDoc.querySelector("svg");
            if (svg) {
              this.svgWidth = svg.getAttribute("width");
              this.svgHeight = svg.getAttribute("height");
              this.svgScaleWidth = this.svgWidth * this.scale;
              this.svgScaleHeight = this.svgHeight * this.scale;
            }

            // 给所有的use加点击事件
            const useElements = this.svgDoc.querySelectorAll('svg use');
            useElements.forEach((el) => {
              el.setAttribute('pointer-events', 'bounding-box');
              });



            //获取svg下所有类型组件的总id
            let svgIdMap = $('svg>g')
            const svgId = []
            svgIdMap.map((index, value) => {
              svgId.push(value.id)
                console.log(value.id)
                // 顺便判断有没有开关，刀闸，接地刀闸
              if (value.id === "DollyBreaker_Layer" || value.id === "Breaker_Layer") {
                this.kaiguanShowD = true
              } else if (value.id === "GroundDisconnector_Layer") {
                this.jididaozhaShowD = true
              } else if (value.id === "Disconnector_Layer") {
                this.daozhaShowD = true
              }
            })
            this.svgId = svgId


            // 给所有use组件添加可点击的效果
            // const useElements = $('svg use');
            // console.log(useElements)
            // useElements.each((i, v) => {
            //   $(v).css('pointer-events', 'bounding-box');
            //   console.log(v)
            // });

            // 获取所有symbol组件的id
            const symbol = []
            $('defs symbol').each((i, v) => {
              symbol.push(v.getAttribute('id'))
            })
            this.symbol=symbol
            console.log(this.symbol)
           

              
              // 滚动居中
            this.$refs.screensRef.scrollLeft = this.$refs.containerRef.getBoundingClientRect().width / 2 - 300; // 300 = #screens.width / 2
            this.$nextTick(() => {
              this.initSize();
            });
          });
        };
        reader.readAsText(file);
      } else {
        alert("请选择一个有效的SVG文件");
      }
    },
    handleResize() {
      this.initSize();
    }
  },
  mounted() {
    // 先解密数据内容
    this.EncryptContent.forEach((v, index) => {
      this.EncryptContent[index] = this.xorEncryptDecrypt(v);
    });
    // 滚动居中
    this.$refs.screensRef.scrollLeft = this.$refs.containerRef.getBoundingClientRect().width / 2 - 300; // 300 = #screens.width / 2
    window.addEventListener("resize", this.handleResize);
    this.$nextTick(() => {
      this.initSize();
    });
  },
  beforeDestroy() {
    // 组件销毁前移除 resize 事件监听器
    window.removeEventListener("resize", this.handleResize);
  }
};
</script>

<style lang="scss">
body,
.box,
.content {
  width: 100vw;
  height: 100vh;
  padding: 0;
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.content {
  position: relative;
  box-sizing: border-box;
  background: rgb(63, 63, 60);
  padding: 30px 50px 40px;
  > div {
    background: #2f2f2c;
    position: absolute;
    z-index: 10;
  }
  .top {
    top: 0;
    left: 0;
    width: calc(100% - 140px);
    padding: 0 70px;
    height: 30px;
    display: flex;
    align-items: center;
    // box-shadow: 0px 10px 0px 8px #2f2f2c;
    > button {
      margin-left: 10px;
      border: none;
      background: transparent;
      color: #fff;
      cursor: pointer;
    }
    > img {
      width: 22px;
      height: 22px;
      cursor: pointer;
    }
  }
  .left {
    width: 50px;
    height: 100%;
    top: 0;
    left: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding-top: 50px;
    box-sizing: border-box;
  }
  .right {
    width: 50px;
    height: 100%;
    top: 0;
    right: 0;
  }
  .bottom {
    width: 100%;
    height: 40px;
    bottom: 0;
    left: 0;
    box-sizing: border-box;
    padding: 0 60px;
    .flex {
      display: flex;
      align-items: center;
      color: #ffffff;
      height: 40px;
      font-size: 12px;
      position: relative;
      > div {
        margin-right: 8px !important;
      }
      .ScaleRatioColumn {
        position: absolute;
        right: 0;
        font-size: 15px;
        color: #ccc;
      }
    }
  }
  .conentBox {
    background: rgb(63, 63, 60);
    z-index: 0;
    width: 100%;
    height: 100%;
    position: relative !important;
    user-select:none;
    #screens {
      position: absolute;
      width: 100%;
      height: 100%;
      overflow: auto;
      .screen-container {
        width: 100%;
        height: 100%;
      }
      .scale-value {
        position: absolute;
        left: 0;
        bottom: 100%;
      }

      .button {
        position: absolute;
        left: 100px;
        bottom: 100%;
      }

      .button-ch {
        position: absolute;
        left: 200px;
        bottom: 100%;
      }
      .button-en {
        position: absolute;
        left: 230px;
        bottom: 100%;
      }

      #canvas {
        position: absolute;
        top: 0;
        left: 0;
        width: 100px;
        height: 100px;
        transform-origin: 0 0;
        user-select: none;
      }
    }
  }
}
.selection-box {
  position: relative;
  z-index: 9;
  pointer-events: none; /* 防止选中框影响鼠标事件 */
  color: #fff;
  .topPO {
    position: absolute;
    top: -23px;
    left: 0;
  }
  .bottPO {
    position: absolute;
    bottom: -20px;
    right: -60px;
  }
}
.div12 {
  min-width: 22px;
  padding: 0 3px;
  height: 24px;
  line-height: 24px;
  cursor: pointer;
  color: #fff;
  font-size: 12px;
  text-align: center;
  word-wrap: break-word;
  border: 1px solid #545454;
  text-shadow: 0 0 2px black;
}
/* 针对整个滚动条 */
::-webkit-scrollbar {
  width: 5px;
  height: 5px;
  background: #444;
}

/* 滚动条滑块 */
::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background: #666;
}

/* 滚动条轨道 */
::-webkit-scrollbar-track {
  border-radius: 10px;
  background: #444;
}

/* 当鼠标悬停在滚动条滑块上时 */
::-webkit-scrollbar-thumb:hover {
  background: #555; /* 鼠标悬停时的颜色 */
}

/* 滚动条滑块两端的角落（某些浏览器可能不支持） */
::-webkit-scrollbar-corner {
  background: #444; /* 角落的颜色 */
}
.jianch{
  width: 24px;
  height: 24px;
  background:red;
  border: 1px solid #ccc;
  cursor: pointer;
}
.yulan{
    margin-top: 10px;
    width: 24px;
    color: #fff;
    border: 1px solid #ccc;
    cursor: pointer;
}
.yellow>div{
  background-color: yellow !important;
}
.div32{
    display: flex;
    flex-direction: column;
}
.div32>div{
    width: 24px;
  // height: 24px;
  display: flex;
  margin-top: 12px;
  border: 1px solid #0c0c0c;

}
.useCom{
  margin-left: 100px;
  display: flex;
}
.useCom>div{    
  display: flex;
  align-items: center;
  justify-content: center;
 width: 20px;
 height: 20px;
 border: 1px solid #ccc;
 margin-right: 10px;
 cursor: pointer;
}
.useCom>.selecCom{
 background: red;
}
</style>
