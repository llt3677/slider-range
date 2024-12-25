<template>
  <!-- 滑块范围选择器容器 -->
  <view class="slider-range" :class="{ disabled }" :style="sliderStyle">
    <view class="slider-range-inner">
      <!-- 滑块条 -->
      <view class="slider-bar">
        <!-- 背景条 -->
        <view class="slider-bar-bg" :style="{ backgroundColor }" />
        <!-- 选中区域条 -->
        <view class="slider-bar-inner" :style="barInnerStyle" />
      </view>

      <!-- 左右两个滑块按钮 -->
      <view
        v-for="block in ['lowerBlock', 'higherBlock']"
        :key="block"
        class="slider-handle-block"
        :style="block === 'lowerBlock' ? lowerHandleStyle : higherHandleStyle"
        @touchstart="onTouchStart"
        @touchmove="onBlockTouchMove"
        @touchend="onBlockTouchEnd"
        @mousedown="onMouseDown"
        :data-tag="block"
      />

      <!-- 滑块值提示 -->
      <view class="range-tip" :style="lowerTipStyle">{{ formatValue(values[0]) }}</view>
      <view class="range-tip" :style="higherTipStyle">{{ formatValue(values[1]) }}</view>

      <!-- 刻度线 -->
      <view
        v-for="n in scaleCount + 1"
        :key="n"
        class="slider-scale"
        :style="{ left: `${(n / scaleCount) * 100}%` }"
      />
      <!-- 最小最大值显示 -->
      <view class="slider-value" style="left: 0">{{ min }}</view>
      <view class="slider-value" style="right: 0">{{ max }}</view>
    </view>
  </view>
</template>

<script>
import throttle from './throttle';
// 默认刻度数量
const DEFAULT_SCALE_COUNT = 24;
// 默认滑块大小(rpx)
const DEFAULT_BLOCK_SIZE = 48;

/**
 * 滑块范围选择器
 * @description 一个可以选择数值范围的滑块组件
 * @tutorial https://ext.dcloud.net.cn/plugin?id=xxx
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

  emits: ['update:modelValue'],

  data() {
    return {
      values: this.modelValue, // 当前选中的值
      startDragPos: 0, // 开始拖动时的位置
      startVal: 0, // 开始拖动时的值
      currentBlock: '', // 当前拖动的滑块
      scaleCount: DEFAULT_SCALE_COUNT, // 刻度数量
      isDragging: false // 是否正在拖动
    };
  },

  computed: {
    // 计算左侧滑块位置
    lowerHandlePosition() {
      return this.calculateHandlePosition(this.values[0]);
    },

    // 计算右侧滑块位置
    higherHandlePosition() {
      return this.calculateHandlePosition(this.values[1]);
    },

    // 左侧滑块样式
    lowerHandleStyle() {
      return this.handleStyle('lowerBlock');
    },

    // 右侧滑块样式
    higherHandleStyle() {
      return this.handleStyle('higherBlock');
    },

    // 左侧提示样式
    lowerTipStyle() {
      return this.tipStyle('lowerBlock');
    },

    // 右侧提示样式
    higherTipStyle() {
      return this.tipStyle('higherBlock');
    },

    // 滑块容器样式
    sliderStyle() {
      const padding = this.blockSize / 2;
      return `padding: 0px ${padding}rpx`;
    },

    // 选中区域样式
    barInnerStyle() {
      const width = ((this.values[1] - this.values[0]) / (this.max - this.min)) * 100;
      return `width: ${width}%;left: ${this.lowerHandlePosition}%;background-color: ${this.activeColor}`;
    }
  },

  watch: {
    // 监听modelValue变化
    modelValue: {
      deep: true,
      immediate: true,
      handler(val) {
        if (!this.valuesEqual(val)) {
          this.updateValues(val);
        }
      }
    }
  },

  methods: {
    // 格式化显示值
    formatValue(val) {
      if (typeof this.format === 'function') {
        return this.format(val);
      }
      return val;
    },

    // 计算滑块位置百分比
    calculateHandlePosition(value) {
      return ((value - this.min) / (this.max - this.min)) * 100;
    },

    // 生成滑块样式
    handleStyle(block) {
      const position = block === 'lowerBlock' ? this.lowerHandlePosition : this.higherHandlePosition;
      let zIndex = this.currentBlock === block ? 20 : 12;

      if ((position < 1 && block === 'lowerBlock') || (position > 99 && block === 'higherBlock')) {
        zIndex = 11;
      }

      return `background-color: ${this.blockColor};width: ${this.blockSize}rpx;height: ${this.blockSize}rpx;left: ${position}%;z-index:${zIndex}`;
    },

    // 生成提示样式
    tipStyle(type) {
      const position = type === 'lowerBlock' ? this.lowerHandlePosition : this.higherHandlePosition;
      const maxDistance = String(this.values[1]).length * 4;
      const distance = maxDistance - (this.higherHandlePosition - this.lowerHandlePosition);

      if (distance > 0) {
        const diff = type === 'lowerBlock' ? -distance : distance;
        return `left: ${position + diff / 2}%`;
      }

      return position < 90
        ? `left: ${position}%`
        : `right: ${100 - position}%; transform: translate(50%, -100%)`;
    },

    // 更新选中值
    updateValues(newVal) {
      if (this.step >= this.max - this.min) {
        throw new RangeError('Invalid slider step or slider range');
      }

      if (!this.isValidValues(newVal)) {
        this.values = [];
        this.$emit('update:modelValue', [], 'update');
        this.$emit('change', []);
        return;
      }

      const newValues = this.calculateNewValues(newVal);
      if (this.valuesEqual(newValues)) return;

      this.values = this.validateValues(newValues);
      this.$emit('update:modelValue', [...this.values], 'update');
      this.$emit('change', [...this.values]);
    },

    // 计算新的值
    calculateNewValues(val) {
      return [
        Math.round((val[0] - this.min) / this.step) * this.step + this.min,
        Math.round((val[1] - this.min) / this.step) * this.step + this.min
      ];
    },

    // 验证并修正值的范围
    validateValues(values) {
      let [lower, higher] = values;

      lower = Math.max(lower, this.min);
      higher = Math.min(higher, this.max);

      if (lower >= higher) {
        if (lower === this.values[0]) {
          higher = lower + this.step;
        } else {
          lower = higher - this.step;
        }
      }

      return [lower, higher];
    },

    // 判断两个值数组是否相等
    valuesEqual(newValues) {
      return Array.isArray(newValues) && 
             Array.isArray(this.values) && 
             newValues.length === this.values.length &&
             newValues.every((val, index) => val === this.values[index]);
    },

    // 开始拖动事件处理
    onTouchStart(event) {
      if (this.disabled) return;

      const tag = event.target.dataset.tag;
      this.currentBlock = tag;
      const { pageX } = event.changedTouches?.[0] || event;
      this.startDragPos = pageX;
      this.startVal = tag === 'lowerBlock' ? this.values[0] : this.values[1];
      this.isDragging = true;
    },

    // 拖动移动事件处理
    onBlockTouchMove(event) {
      if (!this.isDragging || this.disabled) return;
      throttle(this.onDrag(event), 500);
    },

    // 结束拖动事件处理
    onBlockTouchEnd() {
      this.isDragging = false;
    },

    // 拖动处理
    onDrag(event) {
      const view = uni.createSelectorQuery().in(this).select('.slider-range-inner');
      view.boundingClientRect(data => {
        const sliderWidth = data.width;
        const { pageX } = event.changedTouches?.[0] || event;
        const diff = ((pageX - this.startDragPos) / sliderWidth) * (this.max - this.min);
        const nextVal = this.startVal + diff;

        const values = this.currentBlock === 'lowerBlock'
          ? [nextVal, this.values[1]]
          : [this.values[0], nextVal];

        this.updateValues(values);
      }).exec();
    },

    // 验证值是否有效
    isValidValues(values) {
      return Array.isArray(values) && values.length === 2;
    },

    // 添加鼠标按下事件处理
    onMouseDown(event) {
      if (this.disabled) return;
      
      const tag = event.target.dataset.tag;
      this.currentBlock = tag;
      this.startDragPos = event.pageX;
      this.startVal = tag === 'lowerBlock' ? this.values[0] : this.values[1];
      this.isDragging = true;

      // 添加鼠标移动和抬起的事件监听
      document.addEventListener('mousemove', this.onMouseMove);
      document.addEventListener('mouseup', this.onMouseUp);
    },

    // 添加鼠标移动事件处理
    onMouseMove(event) {
      if (!this.isDragging || this.disabled) return;
      event.preventDefault(); // 防止拖动时选中文本
      throttle(this.handleMouseDrag(event), 500);
    },

    // 添加鼠标抬起事件处理
    onMouseUp() {
      this.isDragging = false;
      // 移除事件监听
      document.removeEventListener('mousemove', this.onMouseMove);
      document.removeEventListener('mouseup', this.onMouseUp);
    },

    // 处理鼠标拖动
    handleMouseDrag(event) {
      const view = uni.createSelectorQuery().in(this).select('.slider-range-inner');
      view.boundingClientRect(data => {
        const sliderWidth = data.width;
        const diff = ((event.pageX - this.startDragPos) / sliderWidth) * (this.max - this.min);
        const nextVal = this.startVal + diff;

        const values = this.currentBlock === 'lowerBlock'
          ? [nextVal, this.values[1]]
          : [this.values[0], nextVal];

        this.updateValues(values);
      }).exec();
    }
  }
};
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
