<template>
  <view class="content">
    <!-- 值展示区域 -->
    <view class="value-display">
      <text class="label">当前值:</text>
      <text class="value">{{ rangeValue.join(' - ') }}</text>
    </view>
    
    <!-- 滑块控件 -->
    <view class="slider-container">
      <!-- 由于微信小程序的限制，v-model 无法实时更新，需要使用 model-value 和 change 事件的方式 -->
      <!-- #ifdef MP-WEIXIN -->
      <llt-slider-range 
        :model-value="rangeValue" 
        :min="config.min"
        :max="config.max"
        :step="config.step"
        @change="(val) => rangeValue = val" 
      />
      <!-- #endif -->
      <!-- #ifndef MP-WEIXIN -->
      <llt-slider-range 
        v-model="rangeValue"
        :min="config.min"
        :max="config.max"
        :step="config.step"
      />
      <!-- #endif -->
    </view>

    <!-- 配置选项 -->
    <view class="config-panel">
      <view class="config-item">
        <text class="label">最小值:</text>
        <input type="number" v-model.number="config.min" class="input" />
      </view>
      <view class="config-item">
        <text class="label">最大值:</text>
        <input type="number" v-model.number="config.max" class="input" />
      </view>
      <view class="config-item">
        <text class="label">步长:</text>
        <input type="number" v-model.number="config.step" class="input" />
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      rangeValue: [20, 80],
      config: {
        min: 0,
        max: 100,
        step: 1
      }
    }
  }
}
</script>

<style>
.content {
  padding: 30rpx;
}

.value-display {
  background-color: #f5f5f5;
  padding: 20rpx;
  border-radius: 8rpx;
  margin-bottom: 30rpx;
}

.slider-container {
  margin: 40rpx 0;
}

.config-panel {
  margin-top: 40rpx;
  padding: 20rpx;
  background-color: #f5f5f5;
  border-radius: 8rpx;
}

.config-item {
  display: flex;
  align-items: center;
  margin-bottom: 20rpx;
}

.label {
  font-size: 28rpx;
  color: #666;
  width: 120rpx;
}

.value {
  font-size: 32rpx;
  color: #333;
  font-weight: bold;
}

.input {
  flex: 1;
  height: 60rpx;
  border: 1px solid #ddd;
  border-radius: 6rpx;
  padding: 0 20rpx;
  background-color: #fff;
}
</style>
