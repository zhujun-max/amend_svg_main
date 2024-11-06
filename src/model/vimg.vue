<template>
  <div class="imgBox" ref="maskBox" @mousedown="onmousedownHandle">
    <img
      :src="imageUrl"
      :style="{
        width: imgW + 'px',
        height: imgH + 'px',
        top: top + 'px',
        left: left + 'px',
        transform: scale,
      }"
    />
  </div>
</template>

<script>
export default {
  props: {
    imageSize: {
      type: Object,
      default: () => {},
    },
    imageUrl: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      imgW: 0,
      imgH: 0,
      deg: 0,
      top: 0,
      left: 0,
      scale: "scale(1)",
      size: 0,
      mousewheelevt: null,
    };
  },
  watch: {
    imageSize: {
      handler(v) {
        this.initImage();
      },
      deep: true,
    },
  },

  mounted() {
    //初始化图片
    this.initImage();

    // 兼容火狐浏览器
    this.mousewheelevt = /Firefox/i.test(navigator.userAgent)
      ? "DOMMouseScroll"
      : "mousewheel";
    // 为空间区域绑定鼠标滚轮事件 =》 处理函数是wheelHandle
    // 如果你监听了window的scroll或者touchmove事件，你应该把passive设置为true，这样滚动就会流畅很多
    this.$refs.maskBox.addEventListener(this.mousewheelevt, this.wheelHandle);
  },
  beforeDestroy() {
    //取消监听
    this.$refs.maskBox.removeEventListener(
      this.mousewheelevt,
      this.wheelHandle
    );
  },
  created() {
    // this.handleReset();
  },
  methods: {
    /**
     * 重置
     */
    handleReset() {
      this.imgW = 0;
      this.imgH = 0;
      this.top = 0;
      this.left = 0;
      this.deg = 0;
      this.scale = "scale(1)";
      this.size = 0;
      this.initImage();
    },
    /**
     * 初始化图片
     */
    async initImage() {
      if (!this.imageUrl) {
        return;
      }
      let { width, height } = this.imageSize;
      console.log("解析图片大小", width, height);
      // 设置原始图片的大小
      let realWidth = width;
      let realHeight = height;

      // 获取高宽比例
      const whRatio = realWidth / realHeight;
      const hwRatio = realHeight / realWidth;

      //获取盒子的大小
      const boxW = this.$refs.maskBox.clientWidth;
      const boxH = this.$refs.maskBox.clientHeight;

      if (realWidth >= realHeight) {
        this.imgH = hwRatio * boxW;
        const nih = this.imgH;
        if (nih > boxH) {
          this.imgH = boxH;
          this.imgW = whRatio * boxH;
        } else {
          this.imgW = boxW;
        }
        this.top = (boxH - this.imgH) / 2;
        this.left = (boxW - this.imgW) / 2;
      } else {
        this.imgW = (boxH / realHeight) * realWidth;
        this.imgH = boxH;
        this.left = (boxW - this.imgW) / 2;
      }
    },
    imgScaleHandle(zoom) {
      this.size += zoom;
      if (this.size < -0.5) {
        this.size = -0.5;
      }
      this.scale = `scale(${1 + this.size}) rotateZ(${this.deg}deg)`;
    },

    // 处理图片拖动
    onmousedownHandle(e) {
      const that = this;
      const boxW = this.$refs.maskBox.clientWidth; // 容器的宽度
      const boxH = this.$refs.maskBox.clientHeight; // 容器的高度
      //  console.log('容器。宽:',boxW,'高：',boxH)

      this.$refs.maskBox.onmousemove = function (el) {
        const ev = el || window.event;
        console.log("鼠标移动量。x:", ev.movementX, "y：", ev.movementY);
        ev.preventDefault();

        // 计算缩放后的图片宽高
        const scaledWidth = that.imgW * (1 + that.size);
        const scaledHeight = that.imgH * (1 + that.size);
        
        // 计算新的left和top
        let newLeft = that.left + ev.movementX;
        let newTop = that.top + ev.movementY;
        
        // 计算边界
        const minLeft = Math.min(0, boxW - scaledWidth); // 保证图片右边不超出容器右边界
        const minTop = Math.min(0, boxH - scaledHeight); // 保证图片下边不超出容器下边界
        const maxLeft = Math.max(0, (scaledWidth - boxW) / 2); // 允许图片的左边向右拖拽，直到边缘
        const maxTop = Math.max(0, (scaledHeight - boxH) / 2 +23); // 允许图片的上边向下拖拽，直到边缘
        console.log("最大的top", maxTop, "top：", newTop,'最小',minTop);
        // console.log('计算边界',minLeft,minTop,maxLeft,maxTop)

        // 检查拖拽是否超出边界
        if (newLeft < minLeft) {
          newLeft = minLeft;
        }
        if (newLeft > maxLeft) {
          newLeft = maxLeft;
        }
        if (newTop < minTop) {
          newTop = minTop;
        }
        if (newTop > maxTop) {
          newTop = maxTop;
        }

        // 更新图片位置
        that.left = newLeft;
        that.top = newTop;
      };

      this.$refs.maskBox.onmouseup = function () {
        that.$refs.maskBox.onmousemove = null;
        that.$refs.maskBox.onmouseup = null;
      };

      if (e.preventDefault) {
        e.preventDefault();
      } else {
        return false;
      }
    },

    // 鼠标滚动实现图片缩放
    wheelHandle(e) {
      e.preventDefault();
      const ev = e || window.event;
      const dir = ev.detail ? ev.detail * -120 : ev.wheelDelta;
      this.imgScaleHandle(dir / 200);

      // 缩放后调整图片的left和top，保持图片在容器内
      const scaledWidth = this.imgW * (1 + this.size);
      const scaledHeight = this.imgH * (1 + this.size);
      const boxW = this.$refs.maskBox.clientWidth;
      const boxH = this.$refs.maskBox.clientHeight;

      // 更新边界检查，确保缩放后可以拖动到边缘
      this.left = Math.max(
        Math.min(this.left, Math.max(0, (scaledWidth - boxW) / 2)),
        boxW - scaledWidth
      );
      this.top = Math.max(
        Math.min(this.top, Math.max(0, (scaledHeight - boxH) / 2)),
        boxH - scaledHeight
      );
    },
  },
};
</script>

<style lang="scss" scoped>
.imgBox {
  width: 340px;
  height: 200px;
  overflow: hidden;
  position: relative;
  box-shadow: 0px 0px 4px 0px #ffffff;
  transition: width 1s ease;

  img {
    cursor: move;
    position: absolute;
  }
}

.Tools {
  width: 43px;
  min-height: 100px;
  position: absolute;
  right: -50px;
  top: 0;
  user-select: none;
}
</style>
