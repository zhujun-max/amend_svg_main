<template>
  <div class="box">
    <!-- 内容 -->
    <div class="content">
      <input
        type="file"
        accept=".svg"
        @change="handleFileUpload"
        ref="fileInput"
        style="display: none"
      />
      <!-- top -->
      <div class="top">
        <img
          src="./statc/xiugai.png"
          alt="一键修改"
          title="一键修改"
          @click="basics()"
          class="basicsXiu"
        />
        <button @click="triggerFileUpload">导入</button>
        <button @click="copeSvg()">导出</button>
      </div>
      <!-- left -->
      <div class="left"></div>
      <!-- right -->
      <div class="right"></div>
      <!-- bottom -->
      <div class="bottom">
        <div class="flex">
          <div>组件：</div>
          <div
            v-for="(item, index) in colew"
            :key="index"
            :style="'background-color:' + item"
            class="div12"
            @click="cclier(item)"
          >
            {{ colorClass[item].slice(2, -2) }}
          </div>
          <div>线条：</div>
          <div
            v-for="item in clomne"
            :key="item"
            :style="'background-color:' + item"
            class="div12"
            @click="cclier(item)"
          ></div>
          <div class="ScaleRatioColumn">{{ scale * 100 }}%</div>
        </div>
      </div>
      <!-- 内容 -->
      <div class="conentBox" id="wrapper">
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
          :palette="{
            bgColor: 'rgba(225,225,255, 0)',
            longfgColor: '#bbb',
            shortfgColor: '#bbb',
            fontColor: '#bbb',
            shadowColor: 'red',
            lineColor: '#ccc',
            borderColor: '#00000000',
            cornerActiveColor: 'red',
          }"
        >
        </SketchRule>
        <div
          ref="screensRef"
          id="screens"
          @wheel="handleWheel"
          @scroll="handleScroll"
        >
          <div
            ref="containerRef"
            class="screen-container"
            :style="{
              width: svgScaleWidth + 'px',
              height: svgScaleHeight + 'px',
            }"
            @click="svgClick"
            @mousedown="startSelection"
            @mouseup="updateSelection"
            @mouseover="mouseover"
            @mouseout="mouseout"
            @contextmenu.prevent="handleRightClick"
          >
            <div
              class="selection-box"
              :style="selecFrame.selectionStyle"
              v-if="selecFrame.isSelecting"
            >
              <div class="topPO">
                {{
                  Math.trunc(selecFrame.selectionStyle.left.split("px")[0])
                }}x{{
                  Math.trunc(selecFrame.selectionStyle.top.split("px")[0])
                }}
              </div>
              <div class="bottPO">
                {{
                  Math.trunc(
                    +selecFrame.selectionStyle.left.split("px")[0] +
                      +selecFrame.selectionStyle.width.split("px")[0]
                  )
                }}x{{
                  Math.trunc(
                    +selecFrame.selectionStyle.top.split("px")[0] +
                      +selecFrame.selectionStyle.height.split("px")[0]
                  )
                }}
              </div>
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
      originalText: "svgManualEncryption",
      EncryptContent: [
        "嚎缧凧博禇盩峮隧揋窜诐奤皣覿粋纫",
        "厫畃窾乶頔",
        "儺嬡爫絯彴嚐",
        "仿歗缶纑纲柪",
        "仗皂洦粶纾",
        "收雪弲沯",
        "仸亀剰蠥",
        "昉肋耳勥",
        "坛绉皶見",
        "焔晸揀剻",
        "勛珙皶揪",
        "寺儞陕匦",
        "添附皶見",
        "卉垩畏畸赾茙",
        "寭阳杮勒恚勎",
        "偏玛呪匘",
        "隵揑駱頸",
      ],
      hoveredIndex: null, // 跟踪悬停组件的索引
      svgContent: null,
      newSvgContent: null,
      Svgname: "",
      svgContentSrc: null,
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
          border: "1px solid red",
        },
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
        v: [],
      },
      // 结束
      presetLine: [
        { type: "l", site: 100 },
        { type: "v", site: 200 },
      ],
      // svg名称
      Svgname: "",
      svgContent: null,
      newSvgContent: null,
      Svgname: "",
      svgContentSrc: null,
      svgDoc: "",
      isCtrlPressed: false,
      colew: [
        "rgb(240,65,85)",
        "rgb(128,0,128)",
        "rgb(255,255,0)",
        "rgb(128,128,128)",
        "rgb(0,0,139)",
        "rgb(185,72,66)",
      ],
      clomne: [
        "rgb(0,0,255)",
        "rgb(128,128,128)",
        // "rgb(0,0,0)",
        "rgb(0,255,255)",
      ],
      colorClass: {
        "rgb(240,65,85)": "kv110kV",
        "rgb(128,0,128)": "kv220kV",
        "rgb(255,255,0)": "kv35kV",
        "rgb(128,128,128)": "kv0kV",
        "rgb(0,0,139)": "kv6kV",
        "rgb(185,72,66)": "kv10kV",
      },
      svgWidth: 0,
      svgHeight: 0,
      svgScaleHeight: 0,
      svgScaleWidth: 0,
    };
  },
  components: {
    SketchRule,
  },
  computed: {
    shadow() {
      return {
        x: 0,
        y: 0,
        width: 0,
        height: 0,
      };
    },
    canvasStyle() {
      return {
        width: this.svgWidth + "px",
        height: this.svgHeight + "px",
        transform: `scale(${this.scale})`,
      };
    },
  },
  methods: {
    // 导出svg
    copeSvg() {
    console.log(this.newSvgContent,'this.newSvgContent');
      if (!this.newSvgContent) {
        alert("请先导入svg文件");
        return;
      }
      // 创建一个Blob对象
      const blob = new Blob([this.newSvgContent], {
        type: "image/svg+xml;charset=utf-8",
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
      if (
        event.target.tagName.match(/^(rect|circle|path|use)$/) &&
        !event.target.hasAttribute("initshowingplane")
      ) {
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
      if (
        event.target.tagName.match(/^(rect|circle|path|use)$/) &&
        !event.target.hasAttribute("initshowingplane")
      ) {
        event.target.classList.remove("bordered");
        this.hoveredIndex = null; // 取消悬停
        // event.target.removeAttribute('stroke'); // 或者设置回原来的stroke颜色和宽度
      }
      // console.log('鼠标移出',event)
    },
    // 按下鼠标
    svgClick(event) {
      console.log("按下坐标：：：：：：：：：：", event.target);
    },
    // 按下鼠标左键：
    startSelection(event) {
      console.log(
        "12445按下：",
        event.clientX,
        this.$refs.containerRef.getBoundingClientRect().left
      );

      this.startX =
        event.clientX - this.$refs.containerRef.getBoundingClientRect().left;
      this.startY =
        event.clientY - this.$refs.containerRef.getBoundingClientRect().top;
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
          border: "1px solid red",
        },
      };
    },
    // 鼠标移动，绘制选中框
    updateSelecmove(event) {
      this.selecFrame.isSelecting = true;
      this.endX =
        event.clientX - this.$refs.containerRef.getBoundingClientRect().left;
      this.endY =
        event.clientY - this.$refs.containerRef.getBoundingClientRect().top;
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
            if (
              i === 0 ||
              (parameters[i * 2 - 2] !== currentX &&
                parameters[i * 2 - 1] !== currentY)
            ) {
              // 只有在坐标变化时才添加线段（忽略连续的相同坐标）
              // lineSegments.push({
              //   startX: currentX,
              //   startY: currentY,
              //   endX: parameters[i * 2],
              //   endY: parameters[i * 2 + 1],
              // });
              (lineSegments["startX"] = parameters[i * 2]),
                (lineSegments["startY"] = parameters[i * 2 + 1]);
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
        if (
          isPointInRect(points.startX, points.startY) ||
          isPointInRect(points.endX, points.endY)
        ) {
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
      const wrapperRect = document
        .querySelector("#wrapper")
        .getBoundingClientRect();
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
      const screensRect = document
        .querySelector("#screens")
        .getBoundingClientRect();
      const canvasRect = document
        .querySelector("#canvas")
        .getBoundingClientRect();

      // 标尺开始的刻度
      const startX =
        (screensRect.left + this.thick - canvasRect.left) / this.scale;
      const startY =
        (screensRect.top + this.thick - canvasRect.top) / this.scale;

      this.startX = startX >> 0;
      this.startY = startY >> 0;
    },
    // 选中颜色 修改组件和线条。修改三色组件
    cclier(v) {
      if (this.sanseColor) {
        // 修改三色组件
        // 修改svg组件中的颜色
        this.threeModel.forEach((useElement) => {
          // console.log(useElement)
          // 获取 xlink:href 属性的值
          var xlinkHref = useElement.getAttributeNS(
            "http://www.w3.org/1999/xlink",
            "href"
          );

          // 检查 xlink:href 属性是否存在且以 #Transformer3: 开头
          if (xlinkHref && xlinkHref.startsWith(this.sanseColor)) {
            useElement.setAttribute("class", this.colorClass[v]);
            // console.log("修改三色组件颜色", useElement);
          }
        });
        this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
        // 修改顶部三色组件颜色
        let circle = document.getElementById("sanId");
        const alf = circle.querySelectorAll("use");
        alf.forEach((useElement) => {
          // 获取 xlink:href 属性的值
          var xlinkHref = useElement.getAttributeNS(
            "http://www.w3.org/1999/xlink",
            "href"
          );

          // 检查 xlink:href 属性是否存在且以 #Transformer3: 开头
          if (xlinkHref && xlinkHref.startsWith(this.sanseColor)) {
            // console.log("修改顶部组件的颜色", this.sanseColor, xlinkHref);
            useElement.setAttribute("class", this.colorClass[v]);
          }
        });
        this.sanseColor = "";
      } else {
        // 修改线条颜色
        this.newSelectedLines.forEach((lineId) => {
          lineId.setAttribute("stroke", v); // 改变选中线条的颜色
        });
        // 修改组件颜色
        this.newSelectedModel.forEach((lineId) => {
          lineId.setAttribute("class", this.colorClass[v]); // 改变选中线条的颜色
        });
        this.selecFrame = this.$options.data().selecFrame;

        this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
        this.newSelectedLines = [];
        this.newSelectedModel = [];
      }
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
    // 基础修改
    basics() {
      // 1. 删除选中文字
      const elements1 = this.svgDoc.querySelectorAll("#Text_Layer>text");
      this.EncryptContent.forEach((cc) => {
        elements1.forEach((textElement) => {
          if (cc === textElement.textContent.trim()) {
            // 删除当前标签
            textElement.parentNode.removeChild(textElement);
            return;
          }
        });
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
      const elements4s = this.svgDoc.querySelectorAll(
        "#ACLineSegment_Layer>g>path"
      );
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
      // console.log("删除组件内颜色", elements5);
      elements5.forEach((element) => {
        // 获取stroke属性的内容
        element.setAttribute("stroke", "");
      });

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
        const MeasurementValue_Layer = this.svgDoc.querySelectorAll(
          "#MeasurementValue_Layer text"
        );
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
          if (
            textElement.getAttribute("x") > BackPadd &&
            !this.EncryptContent.includes(textElement.textContent.trim())
          ) {
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
      const aElement = group.querySelector(
        'a[AFMask="39039"]:not([ChangePicPlane])'
      );
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

      this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
    },
    // 文件input调用
    triggerFileUpload() {
      this.$refs.fileInput.click();
    },
    // 修改名称
    repName(filename) {
      console.log("传入的名称", filename);
      // 移除前缀 "CD." 和后缀 ".fac"
      const nameWithoutPrefixSuffix = filename
        .replace(/^CD\./, "")
        .replace(/\.fac/, "");
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
        console.log("获取到svg名称", this.Svgname);
        reader.onload = (e) => {
          this.svgContent = e.target.result;
          this.newSvgContent = e.target.result;
          this.svgContentSrc = URL.createObjectURL(file);
          this.$nextTick(() => {
            const parser = new DOMParser();
            // 解析SVG字符串为DOM对象
            this.svgDoc = parser.parseFromString(
              this.newSvgContent,
              "image/svg+xml"
            );
            const svg = this.svgDoc.querySelector("svg");
            if (svg) {
              this.svgWidth = svg.getAttribute("width");
              this.svgHeight = svg.getAttribute("height");
              this.svgScaleWidth = this.svgWidth * this.scale;
              this.svgScaleHeight = this.svgHeight * this.scale;
            }
            // 滚动居中
            this.$refs.screensRef.scrollLeft =
              this.$refs.containerRef.getBoundingClientRect().width / 2 - 300; // 300 = #screens.width / 2
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
    },
  },
  mounted() {
    // 先解密数据内容
    this.EncryptContent.forEach((v, index) => {
      this.EncryptContent[index] = this.xorEncryptDecrypt(v);
    });
    // 滚动居中
    this.$refs.screensRef.scrollLeft =
      this.$refs.containerRef.getBoundingClientRect().width / 2 - 300; // 300 = #screens.width / 2
    window.addEventListener("resize", this.handleResize);
    this.$nextTick(() => {
      this.initSize();
    });
  },
  beforeDestroy() {
    // 组件销毁前移除 resize 事件监听器
    window.removeEventListener("resize", this.handleResize);
  },
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
  width: 22px;
  height: 22px;
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
</style>
