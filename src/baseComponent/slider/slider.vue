<!------Banner图组件------>
<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup">
      <slot></slot>
    </div>
    <div class="dots">
      <span class="dot" v-for="(item, index) in dots"
      :key="index" :class="{'active' : currentPageIndex === index}"></span>
    </div>
  </div>
</template>
<!------Banner图组件------>

<script>
import BScroll from 'better-scroll'
import { addClass } from 'common/js/dom'
export default {
  data () {
    return {
      dots: [],
      currentPageIndex: 0
    }
  },
  props: {
    // 循环轮播
    loop: {
      type: Boolean,
      default: true
    },
    // 自动轮播
    autoPlay: {
      type: Boolean,
      default: true
    },
    // 间隔
    interval: {
      type: Number,
      default: 4000
    }
  },
  mounted () {
    setTimeout(() => {
      this._setSliderWidth()
      this._initDots()
      this._initSlider()
      this._onScrollEnd()
    }, 20)
    // 页面大小发生变化后重新计算
    window.addEventListener('resize', () => {
      if (!this.slider) {
        return
      }
      this._setSliderWidth(true)
    })
  },
  methods: {
    // 设置轮播图总宽度
    _setSliderWidth () {
      let width = 0
      this.children = this.$refs.sliderGroup.children
      let sliderWidth = this.$refs.slider.clientWidth
      for (let i = 0; i < this.children.length; i++) {
        const child = this.children[i]
        child.style.width = sliderWidth + 'px'
        width += sliderWidth
        addClass(child, 'slider-item')
      }
      if (this.loop) {
        width += 2 * sliderWidth
      }
      this.$refs.sliderGroup.style.width = width + 'px'
    },
    // 初始化轮播参数
    _initSlider () {
      this.slider = new BScroll(this.$refs.slider, {
        scrollX: true,
        // scrollY: false,
        momentum: false,
        snap: {
          loop: this.loop,
          threshold: 0.3,
          speed: 400
        },
        snapSpeed: 400,
        bounce: false,
        stopPropagation: true,
        click: true
      })
      this.slider.on('scrollEnd', this._onScrollEnd)
    },
    _onScrollEnd () {
      let pageIndex = this.slider.getCurrentPage().pageX
      this.currentPageIndex = pageIndex
      if (this.autoPlay) {
        this._play()
      }
    },
    _play () {
      clearTimeout(this.timer)
      this.timer = setTimeout(() => {
        this.slider.next()
      }, this.interval)
    },
    _initDots () {
      this.dots = new Array(this.children.length)
    }
  },
  destroyed () {
    clearTimeout(this.timer)
  }
}
</script>

<style lang="scss" scoped>
  @import "~common/scss/variable.scss";
.slider {
  min-height: 1px;
  position: absolute;
  height: 100%;
  width: 100%;
  top: 0;
  left: 0;
  overflow: hidden;
  border-radius: 8px;
  .slider-group {
    position: relative;
    overflow: hidden;
    white-space: nowrap;
    display: flex;
    height: 100%;
    justify-content: center;
    .slider-item {
      float: left;
      box-sizing: border-box;
      overflow: hidden;
      text-align: center;
      a{
        display: block;
        height: 100%;
      }
      img {
        width: 100%;
        height: 100%;
        display: block;
        border-radius: 8px;
      }
    }
  }
  .dots {
    position: absolute;
    right: 0;
    left: 0;
    bottom: 12px;
    text-align: center;
    height: auto;
    .dot {
      display: inline-block;
      margin: 0 4px;
      width: 4px;
      height: 4px;
      border-radius: 50%;
      background-color: #fff;
      &.active {
        border-radius: 5px;
        background-color: $background-r-color;
      }
    }
  }
}
</style>
