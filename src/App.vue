<template>
  <div class="box">
    <!-- 内容 -->
    <div class="content">
      <input type="file" accept=".svg" @change="handleFileUpload" ref="fileInput" style="display: none" />
      <!-- top -->
      <div class="top" v-if="!Preview">
        <img src="./statc/xiugai.png" alt="一键修改" :disabled="!newSvgContent" title="一键修改" @click="basics()" class="basicsXiu" />
        <button @click="triggerFileUpload()">导入</button>
        <button @click="copeSvg()" :disabled="!newSvgContent">导出</button>
        <button @click="PreviousStep()" :disabled="!svgRecordStack.length">回退至上一步</button>
      </div>
      <!-- left -->
      <div class="left">
        <div @click="RemoveColor()" class="romColor" title="去掉组件默认颜色" :disabled="!newSvgContent" :style="Preview ? 'opacity: 0;cursor: none !important;' : ''">/</div>
        <!-- <div class="jianch" @click="jiancha()">12</div> -->
        <div class="yulan" @click="yulan()" :disabled="!newSvgContent">{{ Preview ? "退出预览" : "预览模式" }}</div>
        <div v-if="Preview" class="div32">
          <div @click="hideSwitch()" :class="hideSwitchShow ? 'backRed' : ''">隐藏接口组件</div>
          <div @click="hideSwitchOther()" :class="hideSwitchOtherShow ? 'backRed' : ''">隐藏接口以外组件</div>
          <div @click="kaiguan" v-if="kaiguanShowD" :class="kaiguanShow ? 'backRed' : ''">开关</div>
          <div @click="jididaozha" v-if="jididaozhaShowD" :class="jididaozhaShow ? 'backRed' : ''">接地刀闸</div>
          <div @click="daozha" v-if="daozhaShowD" :class="daozhaShow ? 'backRed' : ''">刀闸</div>
        </div>
      </div>
      <!-- right -->
      <div class="right"></div>
      <!-- bottom -->
      <div class="bottom">
        <div class="flex" v-if="!Preview">
          <!-- <div>组件：</div> -->
          <div v-for="(item, index) in toolColor" :key="index" :style="'background-color:' + item.color" class="div12" @click="cclier(item)">
            <!-- {{ item.name.match(/\d+/g)[0] }} -->
            {{ item.name.split("kv")[1] }}
          </div>
          <!-- 右边， 点击的组件的所有类型 -->
          <div class="useCom">
            <div v-for="(v, i) in componentType" :key="v" :class="i === componentIndex ? 'selecCom' : ''">
              <div class="use_co" @click="ModifyComponent(i)">{{ v.split(":")[1] }}</div>
              <div class="use_clo" v-if="componentIndex === i">
                <div v-for="(item, i) in toolColor" :key="i" @click="dianji1(item)" :style="'background-color:' + item.color" class="clordr"></div>
                <div class="clordr" @click="dianji1({ color: none })">空</div>
              </div>
            </div>
            <div class="addID" @click="addIDS">+</div>
          </div>
          <div class="ScaleRatioColumn">{{ Math.round(scale * 100) }}%</div>
        </div>
      </div>
      <!-- 内容 -->
      <div class="conentBox" id="wrapper" ref="wrapper">
        <!-- https://github.com/chuxiaoguo/vue-sketch-ruler/blob/master/docs/src/components/UserRuler.vue -->
        <SketchRule :thick="thick" :scale="scale" :width="width" :height="height" :startX="startX" :startY="startY" :shadow="shadow" :horLineArr="lines.h" :verLineArr="lines.v" :cornerActive="true" :isShowRuler="false" :isShowReferLine="false" :palette="palette"> </SketchRule>
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
      palette: {
        bgColor: "rgba(225,225,255, 0)",
        longfgColor: "#bbb",
        shortfgColor: "#bbb",
        fontColor: "#bbb",
        shadowColor: "red",
        lineColor: "#ccc",
        borderColor: "#00000000",
        cornerActiveColor: "red"
      },
      newSvgContentCopy: "",
      Preview: false,
      originalText: "svgManualEncryption",
      EncryptContent: ["嚎缧凧博禇盩峮隧揋窜诐奤皣覿粋纫", "厫畃窾乶頔", "儺嬡爫絯彴嚐", "仿歗缶纑纲柪", "仗皂洦粶纾", "收雪弲沯", "仸亀剰蠥", "昉肋耳勥", "坛绉皶見", "焔晸揀剻", "勛珙皶揪", "寺儞陕匦", "添附皶見", "卉垩畏畸赾茙", "寭阳杮勒恚勎", "偏玛呪匘", "隵揑駱頸", "词奱皶揪粚纱", "隵揑窾", "剅嚈仝", "寒桎仝"],
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
          name: "kv220kV"
        },
        {
          color: "rgb(240,65,85)",
          name: "kv110kV"
        },
        {
          color: "rgb(255,255,0)",
          name: "kv35kV"
        },
        {
          color: "rgb(185,72,66)",
          name: "kv10kV"
        },
        {
          color: "rgb(0,0,139)",
          name: "kv6kV"
        },
        {
          color: "rgb(128,128,128)",
          name: "kv0kV"
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
      hideSwitchOtherShow: false,
      // 存储所有的svg组件的id
      svgId: [],
      // 组件的状态
      kaiguanShow: false,
      daozhaShow: false,
      jididaozhaShow: false,
      kaiguanShowD: false,
      daozhaShowD: false,
      jididaozhaShowD: false,
      symbol: [],
      componentType: [],
      componentIndex: "",
      colorShow: false,
      // 撤回的状态
      chehui: false,
      setObj: {}
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
  watch: {
    newSvgContent: {
      handler(ne, ol) {
        console.log('%c页面数据变动了','background:pink;')
        if (this.chehui) {
          this.chehui = false;
        } else {
          this.svgRecordStack.push(ol);
        }
      },
      deep: true
    },
    newSvgContentCopy: {
      handler(ne, ol) {
        console.log('%c拷贝的数据改变了','background:red;')
      },
      deep: true
    },
  },
  
  methods: {
    // 六位随机数字
    generateSixDigitId() {
      // 创建一个字符数组，包含所有可能的字符（0-9, a-z, A-Z）
      const chars = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
      let uuid = "";

      // 循环32次，每次从字符数组中随机选择一个字符并添加到UUID中
      for (let i = 0; i < 5; i++) {
        const randomIndex = Math.floor(Math.random() * chars.length);
        uuid += chars[randomIndex];
      }

      return uuid;
    },
    // 添加关联数据的方法
    addAssociation(id, data) {
      // 如果id不存在于关联对象中，先创建它并初始化为空数组
      if (!(id in this.setObj)) {
        this.setObj[id] = [];
      }
      // 将数据添加到与该id关联的数组中
      this.setObj[id].push(data);
    },
    // 创建一个新的class，用于同一个组件，多个背景颜色情况。。。。。。。没实现
    addIDS() {
      console.log(this.symbol);
      this.componentType.forEach((id) => {
        const correctRegex = /\:\^/;
        // 如果当前的id就是后续添加的，我就不用克隆了
        if (correctRegex.test(id)) return;
        // 查找原始的symbol元素
        const symbol = this.svgDoc.getElementById(id);
        if (symbol) {
          // 创建一个新的symbol元素的克隆
          const newSymbol = symbol.cloneNode(true);

          // 修改新symbol的ID
          let nameIdS = id.split(":");
          nameIdS[1] = `^${this.generateSixDigitId()}^` + nameIdS[1];
          const nameId = nameIdS.join(":");
          newSymbol.setAttribute("id", nameId);
          console.log(nameId);
          // 将旧名称和新名称导入到关联表中
          this.addAssociation(id, nameId);

          // 创建一个临时的<defs>元素（如果svgContainer中没有的话）
          let defs = this.svgDoc.querySelector("defs");
          if (!defs) {
            defs = this.svgDoc.createElementNS("http://www.w3.org/2000/svg", "defs");
            this.svgDoc.appendChild(defs);
          }

          // 将新克隆的<symbol>添加到<defs>中
          defs.appendChild(newSymbol);
          this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
          console.log("defs::: ", defs);
        }
      });

      this.$nextTick(() => {
        this.componentType = this.componentLI();
      });
    },
    // 根据底部的id名来查找之前添加的有关联的id。
    componentLI() {
      const arrCom = JSON.parse(JSON.stringify(this.componentType));
      this.componentType.forEach((v) => {
        if (this.setObj[v]) {
          arrCom.push(...this.setObj[v]);
        }
      });
      return arrCom;
    },
    // 切换开关的状态（临时观看，不用保存状态，所以用jquery也无所谓）
    kaiguan() {
      this.kaiguanShow = !this.kaiguanShow;
      const LayerG = $("#canvas #DollyBreaker_Layer>g>use");
      LayerG.each((i, v) => {
        const oldxhref = v.getAttribute("xlink:href");
        let newxhref = "";
        if (this.kaiguanShow) {
          if (oldxhref.indexOf("0_0") != -1) {
            newxhref = oldxhref.substring(0, oldxhref.length - 1) + "2";
          } else {
            newxhref = oldxhref.substring(0, oldxhref.length - 1) + "1";
          }
        } else {
          if (oldxhref.indexOf("0_2") != -1) {
            newxhref = oldxhref.substring(0, oldxhref.length - 1) + "0";
          } else {
            newxhref = oldxhref.substring(0, oldxhref.length - 1) + "0";
          }
        }
        v.setAttribute("xlink:href", newxhref);
        // 正常只有这几种状态，其他状态没有做处理
        if (!["1_0_0", "1_0_2", "1_1_0", "1_1_1"].includes(oldxhref.slice(-5))) {
          console.error("竟然有其他开关状态：" + oldxhref);
        }
      });

      const BreakerLayer = $("#canvas #Breaker_Layer>g>use");
      BreakerLayer.each((i, v) => {
        let oldxhref = v.getAttribute("xlink:href");
        let newxhref = "";
        // 正常只有这几种状态，其他状态没有做处理
        if (!["_0", "_1"].includes(oldxhref.slice(-2))) {
          console.error("竟然有其他开关状态：" + oldxhref);
        }
        if (this.kaiguanShow) {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1);
          newxhref = oldxhref + "1";
        } else {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1);
          newxhref = oldxhref + "0";
        }
        v.setAttribute("xlink:href", newxhref);
      });
    },
    // 切换接地刀闸的状态（临时观看，不用保存状态，所以用jquery也无所谓）
    jididaozha() {
      this.jididaozhaShow = !this.jididaozhaShow;
      const GroundDisconnectorG = $("#canvas #GroundDisconnector_Layer>g>use");
      GroundDisconnectorG.each((i, v) => {
        let oldxhref = v.getAttribute("xlink:href");
        let newxhref = "";
        // 正常只有这几种状态，其他状态没有做处理
        if (!["_0", "_1"].includes(oldxhref.slice(-2))) {
          console.error("竟然有其他接地刀闸状态：" + oldxhref);
        }
        if (this.jididaozhaShow) {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1);
          newxhref = oldxhref + "1";
        } else {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1);
          newxhref = oldxhref + "0";
        }
        v.setAttribute("xlink:href", newxhref);
      });
    },
    // 切换刀闸的状态（临时观看，不用保存状态，所以用jquery也无所谓）
    daozha() {
      this.daozhaShow = !this.daozhaShow;
      const DisconnectorG = $("#canvas #Disconnector_Layer>g>use");
      DisconnectorG.each((i, v) => {
        let oldxhref = v.getAttribute("xlink:href");
        let newxhref = "";
        // 正常只有这几种状态，其他状态没有做处理
        if (!["_0", "_1"].includes(oldxhref.slice(-2))) {
          console.error("竟然有其他刀闸状态：" + oldxhref);
        }
        if (this.daozhaShow) {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1);
          newxhref = oldxhref + "1";
        } else {
          oldxhref = oldxhref.substring(0, oldxhref.length - 1);
          newxhref = oldxhref + "0";
        }
        v.setAttribute("xlink:href", newxhref);
      });
    },
    // 隐藏/显示 所有走接口的组件
    hideSwitch() {
      this.hideSwitchShow = !this.hideSwitchShow;
      if (this.hideSwitchShow) {
        $("#DollyBreaker_Layer").hide();
        $("#GroundDisconnector_Layer").hide();
        $("#Disconnector_Layer").hide();
        $("#Breaker_Layer").hide();
      } else {
        $("#DollyBreaker_Layer").show();
        $("#GroundDisconnector_Layer").show();
        $("#Disconnector_Layer").show();
        $("#Breaker_Layer").show();
      }
    },
    // 隐藏走接口组件之外的
    hideSwitchOther() {
      this.hideSwitchOtherShow = !this.hideSwitchOtherShow;
      if (this.hideSwitchOtherShow) {
        this.svgId.forEach((v) => {
          if (!["Head_Layer", "DollyBreaker_Layer", "GroundDisconnector_Layer", "Disconnector_Layer", "Breaker_Layer"].includes(v)) {
            $(`#${v}`).hide();
          }
        });
      } else {
        this.svgId.forEach((v) => {
          if (!["Head_Layer", "DollyBreaker_Layer", "GroundDisconnector_Layer", "Disconnector_Layer", "Breaker_Layer"].includes(v)) {
            $(`#${v}`).show();
          }
        });
      }
    },
    // 点击切换预览模式
    yulan() {
      this.Preview = !this.Preview;
      if (this.Preview) {
        // 深克隆 SVG DOM（深度克隆包括所有子节点）
        this.newSvgContentCopy = new XMLSerializer().serializeToString(this.svgDoc.cloneNode(true));
        // 禁止svg框选。
        this.$refs.containerRef.style.pointerEvents = "none";
        // 关闭之前的框选弹窗
        this.handleRightClick();
      } else {
        // svg还原
        // 多做这一步，时因为直接this.newSvgContent=this.newSvgContentCopy无法触发页面改变，不知道为啥，只能先将数据清空，然后重新赋值
        this.newSvgContent = "";
        const parser = new DOMParser();
        this.svgDoc = parser.parseFromString(this.newSvgContent, "image/svg+xml");
        this.$forceUpdate()
        this.$refs.containerRef.style.pointerEvents = "all";
        this.$nextTick(() => {
          this.newSvgContent=this.newSvgContentCopy 
          this.svgDoc = parser.parseFromString(this.newSvgContent, "image/svg+xml");
          this.hideSwitchShow=false
          this.hideSwitchOtherShow=false
          this.kaiguanShow=false
          this.jididaozhaShow=false
          this.daozhaShow=false
        })
      }
    },
    // 回退至上一步
    PreviousStep() {
      this.chehui = true;
      this.newSvgContent = this.svgRecordStack.pop();
      const parser = new DOMParser();
      this.svgDoc = parser.parseFromString(this.newSvgContent, "image/svg+xml");
    },
    // 导出svg
    copeSvg() {
      if (!this.newSvgContent) {
        alert("请先导入svg文件");
        return;
      }
      let newSvgContentDc = this.newSvgContent;
      const parser = new DOMParser();
      const svgDocD = parser.parseFromString(newSvgContentDc, "image/svg+xml");
      // 如果有当前组件，就去掉之前添加红色块
      const dollyBreakerLayer = svgDocD.getElementById("DollyBreaker_Layer");
      if (dollyBreakerLayer) {
        // 在dollyBreakerLayer上查找所有的rect元素
        const rects = dollyBreakerLayer.querySelectorAll("rect");
        rects.forEach((rectElement) => {
          rectElement.parentNode.removeChild(rectElement);
        });

        // 更新SVG内容
        newSvgContentDc = new XMLSerializer().serializeToString(svgDocD);
      }
      this.$nextTick(() => {
        // 创建一个Blob对象
        const blob = new Blob([newSvgContentDc], {
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
      });
    },
    // 鼠标移入（啥也没干）
    mouseover(event) {
      return;
    },
    // 鼠标移出（啥也没干）
    mouseout(event) {
      return;
    },
    // 按下鼠标
    svgClick(event) {
      // 如果是拖拽，我就清除底部的组件id切换
      this.componentType = [];
      this.componentIndex = "";

      if (!this.selecFrame.isSelecting) {
        // 点击我就清除之前的所有数据
        this.handleRightClick();
      }
      // 如果是点击的组件
      if (Math.trunc(this.selecFrame.selectionStyle.left.split("px")[0]) === Math.trunc(+this.selecFrame.selectionStyle.left.split("px")[0] + +this.selecFrame.selectionStyle.width.split("px")[0]) && Math.trunc(this.selecFrame.selectionStyle.top.split("px")[0]) === Math.trunc(+this.selecFrame.selectionStyle.top.split("px")[0] + +this.selecFrame.selectionStyle.height.split("px")[0])) {
        console.log("点击的同一个位置", event.target);
        const parser = new DOMParser();
        this.svgDoc = parser.parseFromString(this.newSvgContent, "image/svg+xml");
        this.$forceUpdate();
        // 1. 使用svgDoc来修改dom，先找到dom的具体位置。（直接修改dom是无效的，所以通过svgDoc来修改）
        const targetElement = event.target;
        const parentId = targetElement.parentElement ? targetElement.parentElement.id : null;
        if (parentId) {
          // 获取父元素
          const parentElement = this.svgDoc.getElementById(parentId);
          if (parentElement) {
            const xlinkHref = targetElement.getAttribute("xlink:href");
            console.log("id名", xlinkHref);

            if (event.target.tagName === "use") {
              const targetUseElement = Array.from(parentElement.querySelectorAll("use")).find((el) => el.getAttribute("xlink:href") === xlinkHref);
              const href = targetUseElement.getAttribute("xlink:href");
              const hrefSlice = href.slice(1, -1);
              const arrUs = [];
              this.symbol.forEach((v) => {
                if (v.includes(hrefSlice)) {
                  arrUs.push(v);
                }
              });
              this.componentType = arrUs;
              this.componentType = this.componentLI();
              console.log("componentType::: ", this.componentType);
              this.componentIndex = arrUs.findIndex((v) => v === href.slice(1));
              console.log("componentIndex::: ", this.componentIndex);
              this.newSelectedModel = [targetUseElement];
            }
            if (event.target.tagName === "rect" && event.target.getAttribute("xlink:href")) {
              const elementsWithId = this.svgDoc.querySelectorAll(`[id="${parentId}"]`);
              console.log(elementsWithId);
              console.log("id:", parentId);
              // 遍历这些元素
              elementsWithId.forEach((element) => {
                // 检查当前元素是否包含<rect>标签
                const rectElement = element.querySelector("rect");
                if (rectElement) {
                  const parentG = rectElement.parentElement;
                  // 获取所有兄弟use元素（在同一g元素下的其他use元素）
                  const siblingUses = parentG.getElementsByTagName("use")[0];
                  const href = siblingUses.getAttribute("xlink:href");
                  const hrefSlice = href.slice(1, -1);
                  const arrUs = [];
                  this.symbol.forEach((v) => {
                    if (v.includes(hrefSlice)) {
                      arrUs.push(v);
                    }
                  });
                  this.componentType = arrUs;
                  this.componentIndex = arrUs.findIndex((v) => v === href.slice(1));
                  this.newSelectedModel = [siblingUses];
                }
              });
            }

            this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
            // 显示切换组件类型
          }
        }
      }
    },
    // 按下鼠标左键
    startSelection(event) {
      console.log("按下鼠标左键：", event.clientX, this.$refs.containerRef.getBoundingClientRect().left);

      this.startX = event.clientX - this.$refs.containerRef.getBoundingClientRect().left;
      this.startY = event.clientY - this.$refs.containerRef.getBoundingClientRect().top;
      this.selecFrame.isSelecting = false;
      // 绘制选中框
      window.addEventListener("mousemove", this.updateSelecmove);
    },
    // 鼠标右键（主要用于清除框选的区域）
    handleRightClick() {
      console.log("鼠标右键，清除框选状态");
      this.selecFrame.isSelecting = false;
      this.startX = 0;
      this.startY = 0;
      this.end = 0;
      this.endY = 0;
      this.newSelectedLines = [];
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
    // 鼠标移动，绘制选中框（获取起点坐标和结尾坐标）
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
      this.newSelectedLines = this.getIntersectingLines(left, top, Mleft, Mtop);
      this.newSelectedModel = this.getIntersectingmodel(left, top, Mleft, Mtop);
      window.removeEventListener("mousemove", this.updateSelecmove);
    },
    // 修改组件的xlink:href（类似切换开合状态）
    ModifyComponent(v) {
      if (this.newSelectedModel.length === 1) {
        this.newSelectedModel[0].setAttribute("xlink:href", "#" + this.componentType[v]);
        this.componentIndex = v;
        this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
      }
    },
    // 返回选中区域的组件
    getIntersectingmodel(startX, startY, endX, endY) {
      const intersectin = [];
      // 1. 拿到所有的线条
      const Model = this.svgDoc.querySelectorAll("use");
      // 定义一个函数来检查点是否在矩形内
      function isPointInRect(x, y) {
        return x >= startX && x <= endX && y >= startY && y <= endY;
      }
      // 定义一个函数来检查线段是否与矩形相交
      const doesLineIntersectRect = (v) => {
        // 获取线段的两个端点坐标。并且还要*缩放比例
        const sx = parseInt(v.getAttribute("x")) * this.scale;
        const sy = parseInt(v.getAttribute("y")) * this.scale;
        const ex = sx + parseInt(v.getAttribute("width")) * this.scale;
        const ey = sy + parseInt(v.getAttribute("height")) * this.scale;

        // 检查线段的两个端点是否在矩形内
        if (isPointInRect(sx, sy) || isPointInRect(ex, ey)) {
          return true; // 如果一个端点在矩形内，则线段与矩形相交
        }
        return false;
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
        const lineSegments = {};
        let currentX = 0,
          currentY = 0;

        for (let i = 0; i < commands.length; i++) {
          const cmd = commands[i];
          let nextX, nextY;

          if (cmd === "M") {
            // moveto 命令设置新的当前点
            if (i === 0 || (parameters[i * 2 - 2] !== currentX && parameters[i * 2 - 1] !== currentY)) {
              (lineSegments["startX"] = parameters[i * 2]), (lineSegments["startY"] = parameters[i * 2 + 1]);
            }
            currentX = parameters[i * 2] * this.scale;
            currentY = parameters[i * 2 + 1] * this.scale;
          } else if (cmd === "L") {
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
    // 获取div的大小(用于标尺的宽高)
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
    // 标尺开始的刻度 页面宽高变化就要更新
    handleScroll() {
      const screensRect = document.querySelector("#screens").getBoundingClientRect();
      const canvasRect = document.querySelector("#canvas").getBoundingClientRect();

      const startX = (screensRect.left + this.thick - canvasRect.left) / this.scale;
      const startY = (screensRect.top + this.thick - canvasRect.top) / this.scale;

      this.startX = startX >> 0;
      this.startY = startY >> 0;
    },
    // 修改组件默认颜色，（改组件填充色）
    dianji1(v) {
      if (this.newSelectedModel.length === 1) {
        console.log(this.componentType[this.componentIndex]);
        const symbol = this.svgDoc.getElementById(this.componentType[this.componentIndex]);
        if (symbol) {
          const rect = symbol.querySelector("rect");
          if (rect) {
            rect.setAttribute("fill", v.color);
            this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
          }
        } else {
          console.error("没有找到我需要的组件，不应该呀");
        }
      }
    },
    // 选中颜色 修改组件和线条。修改三色组件
    cclier(v) {
      console.log("修改线条颜色::: ", this.newSelectedLines);
      console.log("修改组件颜色::: ", this.newSelectedModel);
      // 修改线条颜色
      this.newSelectedLines.forEach((lineId) => {
        lineId.setAttribute("stroke", v.color);
      });
      // 修改组件颜色
      this.newSelectedModel.forEach((lineId) => {
        lineId.setAttribute("class", v.name);
      });
      // 如果是点击的组件，那么只有一个，就去掉颜色
      if (this.newSelectedModel.length===1) {
        const hrefID = this.newSelectedModel[0].getAttribute('xlink:href')
        const selector = hrefID.split("#")[1]
        const symbolElement = this.svgDoc.getElementById(selector);
        console.log(symbolElement)
        if (symbolElement) {
          const childElements = symbolElement.querySelectorAll('*');
          childElements.forEach(element => {
            element.removeAttribute('stroke');
          });
        }
        
      }
      // 修改组件的颜色，查看组件的symbol内默认颜色是否已经去掉，没有我就重新去一遍
      
      this.selecFrame = this.$options.data().selecFrame;
      this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
      this.newSelectedLines = [];
      this.newSelectedModel = [];
    },
    // 无需多言
    xorEncryptDecrypt(text) {
      let keyChars = this.originalText.split("");
      let keyIndex = 0;
      let textChars = text.split("");
      let result = "";
      for (let i = 0; i < textChars.length; i++) {
        let charCode = textChars[i].charCodeAt(0);
        let keyCharCode = keyChars[keyIndex % keyChars.length].charCodeAt(0);
        let xorResult = charCode ^ keyCharCode;
        result += String.fromCharCode(xorResult);
        keyIndex++;
      }
      return result;
    },
    // 一键修改
    basics() {
      // 1. 删除选中文字
      const elements1 = this.svgDoc.querySelectorAll("#Text_Layer>text");
      // 存储所有匹配到文本的y轴（用于删除没有匹配到的文本，根据当前y轴来判断）
      const textY = [];
      this.EncryptContent.forEach((cc) => {
        elements1.forEach((textElement) => {
          // 只要文本中包含该文字
          if (textElement.textContent.trim().includes(cc)) {
            // 如果已经被删除，就跳过
            if (!textElement.parentNode) return;
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

      // 2.删除所有图片标签
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

      // 5. 给所有的use加点击事件（由于所有组件只能点击边框才行，实在不方便）
      const useElements = this.svgDoc.querySelectorAll("svg use");
      useElements.forEach((el) => {
        el.setAttribute("pointer-events", "bounding-box");
      });
      // 给DollyBreaker_Layer组件的外层添加可点击的事件（将DollyBreaker_Layer标签外层的div的pointer-events去掉，不然会影响内部元素的点击）
      let $gs = this.svgDoc.querySelectorAll("#DollyBreaker_Layer g");
      // 创建一个map来存储每个id出现的次数
      var idCountMap = {};
      $gs.forEach((el) => {
        var id = el.getAttribute("id");
        if (idCountMap[id]) {
          idCountMap[id]++;
        } else {
          idCountMap[id] = 1;
        }
        // 如果是相同id的第二个g标签
        if (idCountMap[id] === 2) {
          var $use = el.querySelectorAll("use")[0];
          // 移除pointer-events属性
          $use.removeAttribute("pointer-events");

          // 创建rect元素，完全复制一份use组件，为了实现外层组件可点击，用于修改外层组件的样式。
          const rect = this.svgDoc.createElementNS("http://www.w3.org/2000/svg", "rect");
          rect.setAttribute("x", $use.getAttribute("x"));
          rect.setAttribute("y", $use.getAttribute("y"));
          rect.setAttribute("width", $use.getAttribute("width"));
          rect.setAttribute("height", "16");
          rect.setAttribute("transform", $use.getAttribute("transform"));
          rect.setAttribute("xlink:href", $use.getAttribute("xlink:href"));
          rect.setAttribute("pointer-events", "bounding-box");
          // rect.setAttribute("class", $use.getAttribute("class"));
          // 给外层的use添加一个颜色，是方便后续看到好点击，导出的时候会把这个rect去掉
          rect.setAttribute("fill", "#bc1111a3");
          el.appendChild(rect);
        }
      });

      // 6.删除可以跳转的标签，删除a标签的xlink:href属性
      const hrefSkip = this.svgDoc.querySelectorAll("a");
      hrefSkip.forEach((v) => {
        v.removeAttribute("xlink:href");
      });

      // 7. 删除最底部的线条
      const bottomLine = this.svgDoc.querySelectorAll("line");
      const backHeight = this.svgDoc.querySelector("#Head_Layer>rect");
      const Backhei = parseInt(backHeight.getAttribute("height"));
      bottomLine.forEach((v) => {
        if (Backhei - parseInt(v.getAttribute("y2")) < 40) {
          v.parentNode.removeChild(v);
        }
      });

      // 9. 删除右边的内容。右下角的点点点的。并加文本
      const GZPLayer = this.svgDoc.getElementById("GZP_Layer");
      if (GZPLayer) {
        // 9.1
        const BackW = backHeight.getAttribute("width");
        // 获得文字的左右边距，
        const BackPadd = BackW - 500;
        // 删除数字
        const MeasurementValue_Layer = this.svgDoc.querySelectorAll("#MeasurementValue_Layer text");
        MeasurementValue_Layer.forEach((textElement) => {
          if (textElement.getAttribute("x") > BackPadd) {
            // 删除当前标签
            textElement.parentNode.removeChild(textElement);
          }
        });
        // 删除文字
        elements1.forEach((textElement) => {
          if (textElement.getAttribute("x") > BackPadd && !this.EncryptContent.includes(textElement.textContent.trim())) {
            // 删除当前标签
            textElement.parentNode.removeChild(textElement);
          }
        });

        // 9.2删除点点点
        GZPLayer.parentNode.removeChild(GZPLayer);
      }

      // 10. 删除值班名单下的背景框
      const group = this.svgDoc.getElementById("Other_Layer");
      // 先将39039作为匹配项
      const aElement = group.querySelector('a[AFMask="39039"]:not([ChangePicPlane])');
      if (aElement) {
        const firstRect = aElement.querySelector("rect");
        if (firstRect) {
          firstRect.parentNode.removeChild(firstRect);
        }
      } else {
        // 使用第二种方法
        const aElement2 = group.querySelectorAll("a:not([ChangePicPlane])");
        aElement2.forEach(function (va) {
          const rect = va.querySelector("rect");
          // 直接将颜色作为匹配项
          if (rect.getAttribute("fill") === "rgb(0,85,0)") {
            if (parseInt(rect.getAttribute("x"), 10) < 100) {
              // 如果满足条件，则可以进行一些操作，比如改变其样式
              rect.parentNode.removeChild(rect);
            }
          }
        });
      }

      // 做完修改dom操作，一定要解析为字符串，好渲染到页面中
      this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
    },
    // 删除组件内的颜色（除开圆点图案）
    RemoveColor() {
      const elements5 = this.svgDoc.querySelectorAll("defs>symbol>*");
      const Protect_LayerUse = this.svgDoc.querySelectorAll("#Protect_Layer>g>use");
      let circleName = [];
      Protect_LayerUse.forEach((v) => {
        circleName.push(v.getAttribute("xlink:href"));
      });
      circleName = [...new Set(circleName)];
      const circleNa = circleName.map((v) => {
        v = v.split("#")[1];
        return v;
      });
      if (circleNa.length) {
        circleNa.forEach((ecve) => {
          elements5.forEach((element) => {
            // 获取stroke属性的内容
            if (ecve !== element.parentNode.getAttribute("id")) {
              element.setAttribute("stroke", "");
            }
            // 点点点的默认组件颜色还不能删
          });
        });
      } else {
        elements5.forEach((element) => {
          element.setAttribute("stroke", "");
        });
      }
      // 做完修改dom操作，一定要解析为字符串，好渲染到页面中
      this.newSvgContent = new XMLSerializer().serializeToString(this.svgDoc);
    },

    // 顶部和底部名称删除 顶部和底部分成两个数组。
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
    // 顶部和底部名称删除 误差对比
    contrastArr(y, arr, offset) {
      const numbers = arr.map(Number);
      const target = Number(y);
      // 检查数组中的每个数字是否在范围内
      for (let num of numbers) {
        if (Math.abs(num - target) <= offset) {
          return true;
        }
      }
      return false;
    },
    // 文件input调用
    triggerFileUpload() {
      this.$refs.fileInput.click();
    },
    // 修改名称
    repName(filename) {
      // 移除前缀 "CD." 和后缀 ".fac"
      const nameWithoutPrefixSuffix = filename.replace(/^CD\./, "").replace(/\.fac/, "");
      // 将下划线改为.
      const newName = `${nameWithoutPrefixSuffix.replace(/_/g, ".")}`;
      return newName;
    },
    // 导入svg
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file && file.type === "image/svg+xml") {
        const reader = new FileReader();
        // 报名文件名，用于下载
        this.Svgname = this.repName(file.name);
        reader.onload = (e) => {
          // svg，在页面使用v-html渲染。（切记：不能直接通过获取dom来修改元素，不然状态保持不住）
          this.newSvgContent = e.target.result;
          this.svgRecordStack = [];
          this.$nextTick(() => {
            const parser = new DOMParser();
            // 解析SVG字符串为DOM对象（切记：如果要修改dom，只能通过修改svgDoc来修改dom，还需要将svgDoc转为字符串=newSvgContent）
            this.svgDoc = parser.parseFromString(this.newSvgContent, "image/svg+xml");

            // 以下操作，都不会修改元素

            // 1. 获取svg宽高尺寸。不修改元素
            const svg = this.svgDoc.querySelector("svg");
            if (svg) {
              this.svgWidth = svg.getAttribute("width");
              this.svgHeight = svg.getAttribute("height");
              this.svgScaleWidth = this.svgWidth * this.scale;
              this.svgScaleHeight = this.svgHeight * this.scale;
            }

            // 2. 获取svg下所有类型组件的id。不修改元素
            let svgIdMap = this.svgDoc.querySelectorAll("svg>g");
            const svgId = [];
            svgIdMap.forEach((value) => {
              svgId.push(value.id);
              // 顺便判断有没有开关，刀闸，接地刀闸
              if (value.id === "DollyBreaker_Layer" || value.id === "Breaker_Layer") {
                this.kaiguanShowD = true;
              } else if (value.id === "GroundDisconnector_Layer") {
                this.jididaozhaShowD = true;
              } else if (value.id === "Disconnector_Layer") {
                this.daozhaShowD = true;
              }
            });
            this.svgId = svgId;

            // 3. 获取所有symbol组件的id。不修改元素
            const symbol = [];
            this.svgDoc.querySelectorAll("defs symbol").forEach((v) => {
              symbol.push(v.getAttribute("id"));
            });
            this.symbol = symbol;

            // 4. svg靠左上角对齐
            this.$refs.screensRef.scrollLeft = 0; // 滚动到最左边
            this.$refs.screensRef.scrollTop = 0; // 滚动到顶部

            // 5. 修改尺子的尺寸
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
    // svg靠左上角对齐
    this.$refs.screensRef.scrollLeft = 0;
    this.$refs.screensRef.scrollTop = 0;
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
    background: rgb(63, 63, 60) !important;
    z-index: 0;
    width: 100%;
    height: 100%;
    position: relative !important;
    user-select: none;
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
.jianch {
  width: 24px;
  height: 24px;
  background: red;
  border: 1px solid #ccc;
  cursor: pointer;
}
.yulan {
  text-align: center;
  font-size: 14px;
  margin-top: 10px;
  width: 24px;
  color: #fff;
  border: 1px solid #ccc;
  cursor: pointer;
  user-select: none;
}
.div32 {
  display: flex;
  flex-direction: column;
}
.div32 > div {
  text-align: center;
  font-size: 14px;
  width: 24px;
  // height: 24px;
  display: flex;
  margin-top: 12px;
  border: 1px solid #ccc;
  cursor: pointer;
  user-select: none;
  color: #fff;
}
.useCom {
  margin-left: 100px;
  display: flex;
}
.useCom > div {
  display: flex;
  margin-right: 15px;
  align-items: center;
  justify-content: center;
}
.useCom .use_co {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 5px;
  height: 24px;
  border: 1px solid #ccc;
  cursor: pointer;
}
.useCom .selecCom {
  background: red;
}

.clordr {
  width: 24px;
  height: 24px;
  border: 1px solid #2f2f2c;
  display: flex;
  align-items: center;
  justify-content: center;
}
.use_clo {
  display: flex;
  border: 1px solid #ccc;
}
.addID {
  width: 20px;
  height: 20px;
  border: 1px solid #ccc;
  cursor: pointer;
}
.romColor {
  border: 1px solid #ccc;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: red;
  color: #ffffff;
  font-size: 15px;
  cursor: pointer;
}
.backRed {
  background: red;
}
[disabled="disabled"] {
  opacity: 0.5;
  cursor: no-drop !important;
  pointer-events: none;
}
</style>
