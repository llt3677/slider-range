<template>
  <view class="content">
    <view style="margin-bottom: 20rpx;">
      {{ rangeValue }}
    </view>
    <llt-slider-range v-model="rangeValue" />
  </view>
</template>

<script>
export default {
  data() {
    return {
      rangeValue: [20, 80]
    }
  }
}
</script>

<style>
	.content {
		padding: 60rpx;
	}

</style>
