<template>
  <!-- 滑块范围选择器容器 -->
  <view class="slider-range" :class="{disabled}" :style="sliderStyle">
    <view class="slider-range-inner">
      <!-- 滑块条 -->
      <view class="slider-bar">
        <!-- 背景条 -->
        <view class="slider-bar-bg" :style="{backgroundColor}" />
        <!-- 选中区域条 -->
        <view class="slider-bar-inner" :style="barInnerStyle" />
      </view>

      <!-- 左右两个滑块按钮 -->
      <view
        v-for="block in ['lowerBlock', 'higherBlock']"
        :key="block"
        class="slider-handle-block"
        :style="block === 'lowerBlock' ? leftHandleStyle : rightHandleStyle"
        :data-tag="block"
        @touchstart="handleDragStart"
        @touchmove="handleDragMove"
        @touchend="onBlockTouchEnd"
        @mousedown="onMouseDown"
      />

      <!-- 滑块值提示 -->
      <view class="range-tip" :style="leftTipStyle">{{ formatValue(selectedRange[0]) }}</view>
      <view class="range-tip" :style="rightTipStyle">{{ formatValue(selectedRange[1]) }}</view>

      <!-- 刻度线 -->
      <view
        v-for="n in scaleCount + 1"
        :key="n"
        class="slider-scale"
        :style="{left: `${(n / scaleCount) * 100}%`}"
      />
      <!-- 最小最大值显示 -->
      <view class="slider-value" style="left: 0">{{ min }}</view>
      <view class="slider-value" style="right: 0">{{ max }}</view>
    </view>
  </view>
</template>

<script>
import throttle from './throttle'
// 默认刻度数量
const DEFAULT_SCALE_COUNT = 24
// 默认滑块大小(rpx)
const DEFAULT_BLOCK_SIZE = 48

/**
 * 滑块范围选择器
 * @description 一个可以选择数值范围的滑块组件
 * @tutorial https://ext.dcloud.net.cn/plugin?id=21575
 * @property {Array} modelValue 双向绑定的值，默认[0, 100]
 * @property {Number} min 最小值，默认0
 * @property {Number} max 最大值，默认100
 * @property {Number} step 步长，默认1
 * @property {Function} format 格式化显示的值的函数
 * @property {Boolean} disabled 是否禁用，默认false
 * @property {String} backgroundColor 背景颜色，默认#F6F6F6
 * @property {String} activeColor 激活颜色，默认#4DB8F6
 * @property {Number} blockSize 滑块大小，默认48
 * @property {String} blockColor 滑块颜色，默认#fff
 * @event {Function} update:modelValue 值变化时触发
 */

export default {
  name: 'llt-slider-range',
  // 支持v-model双向绑定
  model: {
    prop: 'modelValue',
    event: 'update:modelValue'
  },
  props: {
    modelValue: {
      type: Array,
      default: () => [0, 100]
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 100
    },
    step: {
      type: Number,
      default: 1
    },
    format: {
      type: Function,
      default: val => val
    },
    disabled: {
      type: Boolean,
      default: false
    },
    backgroundColor: {
      type: String,
      default: '#F6F6F6'
    },
    activeColor: {
      type: String,
      default: '#4DB8F6'
    },
    blockSize: {
      type: Number,
      default: DEFAULT_BLOCK_SIZE
    },
    blockColor: {
      type: String,
      default: '#fff'
    }
  },

  emits: ['update:modelValue', 'change'],

  data() {
    return {
      selectedRange: this.modelValue, // 当前选中的值
      dragStartPosition: 0, // 开始拖动时的位置
      dragStartValue: 0, // 开始拖动时的值
      activeBlock: '', // 当前拖动的滑块
      scaleCount: DEFAULT_SCALE_COUNT, // 刻度数量
      isDragging: false // 是否正在拖动
    }
  },

  computed: {
    // 计算左侧滑块位置
    leftHandlePosition() {
      return this.calculateHandlePosition(this.selectedRange[0])
    },

    // 计算右侧滑块位置
    rightHandlePosition() {
      return this.calculateHandlePosition(this.selectedRange[1])
    },

    // 左侧滑块样式
    leftHandleStyle() {
      return this.generateHandleStyle('lowerBlock')
    },

    // 右侧滑块样式
    rightHandleStyle() {
      return this.generateHandleStyle('higherBlock')
    },

    // 左侧提示样式
    leftTipStyle() {
      return this.generateTipStyle('lowerBlock')
    },

    // 右侧提示样式
    rightTipStyle() {
      return this.generateTipStyle('higherBlock')
    },

    // 滑块容器样式
    sliderStyle() {
      const padding = this.blockSize / 2
      return `padding-left: ${padding}rpx;padding-right: ${padding}rpx`
    },

    // 选中区域样式
    barInnerStyle() {
      const width = ((this.selectedRange[1] - this.selectedRange[0]) / (this.max - this.min)) * 100
      return `width: ${width}%;left: ${this.leftHandlePosition}%;background-color: ${this.activeColor}`
    }
  },

  watch: {
    // 监听modelValue变化
    modelValue: {
      deep: true,
      immediate: true,
      handler(val) {
        if (!this.valuesEqual(val)) {
          this.updateValues(val)
        }
      }
    }
  },

  methods: {
    // 格式化显示值
    formatValue(val) {
      if (typeof this.format === 'function') {
        return this.format(val)
      }
      return val
    },

    // 计算滑块位置百分比
    calculateHandlePosition(value) {
      return ((value - this.min) / (this.max - this.min)) * 100
    },

    // 生成滑块样式
    generateHandleStyle(block) {
      const position = block === 'lowerBlock' ? this.leftHandlePosition : this.rightHandlePosition
      let zIndex = this.activeBlock === block ? 20 : 12

      if ((position < 1 && block === 'lowerBlock') || (position > 99 && block === 'higherBlock')) {
        zIndex = 11
      }

      return `background-color: ${this.blockColor};width: ${this.blockSize}rpx;height: ${this.blockSize}rpx;left: ${position}%;z-index:${zIndex}`
    },

    // 生成提示样式
    generateTipStyle(type) {
      const position = type === 'lowerBlock' ? this.leftHandlePosition : this.rightHandlePosition
      // 计算最大显示距离,根据右侧值的字符长度乘以8得到基准距离
      const maxDistance = String(this.selectedRange[1]).length * 8
      // 计算实际距离,用最大距离减去两个滑块之间的距离
      const distance = maxDistance - (this.rightHandlePosition - this.leftHandlePosition)

      // 如果实际距离大于0,说明两个滑块太近,需要调整提示位置避免重叠
      if (distance > 0) {
        // 根据滑块类型计算偏移量,左滑块向左偏移,右滑块向右偏移
        const diff = type === 'lowerBlock' ? -distance : distance
        return `left: calc(${position}% + ${diff}rpx)`
      }

      return position < 90
        ? `left: ${position}%`
        : `right: ${100 - position}%; transform: translate(50%, -100%)`
    },

    // 更新选中值
    updateValues(newVal) {
      if (this.step >= this.max - this.min) {
        throw new RangeError('Invalid slider step or slider range')
      }

      if (!this.isValidValues(newVal)) {
        this.selectedRange = []
        this.$emit('update:modelValue', [], 'update')
        this.$emit('change', [])
        return
      }

      const newValues = this.calculateNewValues(newVal)
      if (this.valuesEqual(newValues)) return

      this.selectedRange = this.validateValues(newValues)
      this.$emit('update:modelValue', [...this.selectedRange], 'update')
      this.$emit('change', [...this.selectedRange])
    },

    // 计算新的值
    calculateNewValues(val) {
      return [
        Math.round((val[0] - this.min) / this.step) * this.step + this.min,
        Math.round((val[1] - this.min) / this.step) * this.step + this.min
      ]
    },

    // 验证并修正值的范围
    validateValues(values) {
      let [lower, higher] = values

      lower = Math.max(lower, this.min)
      higher = Math.min(higher, this.max)

      if (lower >= higher) {
        if (lower === this.selectedRange[0]) {
          higher = lower + this.step
        } else {
          lower = higher - this.step
        }
      }

      return [lower, higher]
    },

    // 判断两个值数组是否相等
    valuesEqual(newValues) {
      return Array.isArray(newValues) && 
        Array.isArray(this.selectedRange) && 
        newValues.length === this.selectedRange.length &&
        newValues.every((val, index) => val === this.selectedRange[index])
    },

    // 开始拖动事件处理
    handleDragStart(event) {
      if (this.disabled) return

      const tag = event.target.dataset.tag
      this.activeBlock = tag
      const { pageX } = event.changedTouches?.[0] || event
      this.dragStartPosition = pageX
      this.dragStartValue = tag === 'lowerBlock' ? this.selectedRange[0] : this.selectedRange[1]
      this.isDragging = true
    },

    // 拖动移动事件处理
    handleDragMove(event) {
      if (!this.isDragging || this.disabled) return
      throttle(this.processDrag(event), 500)
    },

    // 结束拖动事件处理
    onBlockTouchEnd() {
      this.isDragging = false
    },

    // 拖动处理
    processDrag(event) {
      const view = uni.createSelectorQuery().in(this).select('.slider-range-inner')
      view.boundingClientRect(data => {
        const sliderWidth = data.width
        const { pageX } = event.changedTouches?.[0] || event
        const diff = ((pageX - this.dragStartPosition) / sliderWidth) * (this.max - this.min)
        const nextVal = this.dragStartValue + diff

        const values = this.activeBlock === 'lowerBlock'
          ? [nextVal, this.selectedRange[1]]
          : [this.selectedRange[0], nextVal]

        this.updateValues(values)
      }).exec()
    },

    // 验证值是否有效
    isValidValues(values) {
      return Array.isArray(values) && values.length === 2
    },

    // 添加鼠标按下事件处理
    onMouseDown(event) {
      if (this.disabled) return
      
      const tag = event.target.dataset.tag
      this.activeBlock = tag
      this.dragStartPosition = event.pageX
      this.dragStartValue = tag === 'lowerBlock' ? this.selectedRange[0] : this.selectedRange[1]
      this.isDragging = true

      // 添加鼠标移动和抬起的事件监听
      document.addEventListener('mousemove', this.onMouseMove)
      document.addEventListener('mouseup', this.onMouseUp)
    },

    // 添加鼠标移动事件处理
    onMouseMove(event) {
      if (!this.isDragging || this.disabled) return
      event.preventDefault() // 防止拖动时选中文本
      throttle(this.handleMouseDrag(event), 500)
    },

    // 添加鼠标抬起事件处理
    onMouseUp() {
      this.isDragging = false
      // 移除事件监听
      document.removeEventListener('mousemove', this.onMouseMove)
      document.removeEventListener('mouseup', this.onMouseUp)
    },

    // 处理鼠标拖动
    handleMouseDrag(event) {
      const view = uni.createSelectorQuery().in(this).select('.slider-range-inner')
      view.boundingClientRect(data => {
        const sliderWidth = data.width
        const diff = ((event.pageX - this.dragStartPosition) / sliderWidth) * (this.max - this.min)
        const nextVal = this.dragStartValue + diff

        const values = this.activeBlock === 'lowerBlock'
          ? [nextVal, this.selectedRange[1]]
          : [this.selectedRange[0], nextVal]

        this.updateValues(values)
      }).exec()
    }
  }
}
</script>

<style lang="scss" scoped>
.slider-range {
  position: relative;
  padding-top: 40rpx;

  &-inner {
    position: relative;
    width: 100%;
    height: 100rpx;
  }

  &.disabled {
    .slider-bar-inner {
      opacity: 0.35;
    }

    .slider-handle-block {
      cursor: not-allowed;
    }
  }
}

.slider-bar {
  position: absolute;
  top: 30%;
  left: 0;
  right: 0;
  height: 15rpx;
  transform: translateY(-30%);

  &-inner,
  &-bg {
    position: absolute;
    width: 100%;
    height: 100%;
    border-radius: 10000px;
  }

  &-inner {
    z-index: 11;
  }

  &-bg {
    z-index: 10;
  }
}

.slider-handle-block {
  position: absolute;
  top: 30%;
  transform: translate(-50%, -50%);
  border-radius: 50%;
  box-shadow: 0rpx 0rpx 10rpx 0rpx rgba(91, 91, 91, 0.2);
  z-index: 12;
  cursor: pointer;
  user-select: none;
}

.range-tip {
  position: absolute;
  top: 0;
  font-family: Source Han Sans CN;
  font-weight: 400;
  font-size: 26rpx;
  color: #666666;
  transform: translate(-30%, -100%);
}

.slider-scale {
  position: absolute;
  bottom: 30rpx;
  width: 1rpx;
  height: 14rpx;
  background: #e2e2e2;
}

.slider-value {
  position: absolute;
  bottom: 0;
  font-family: Source Han Sans CN;
  font-weight: 400;
  font-size: 21rpx;
  color: #bbbbbb;
}
</style>
