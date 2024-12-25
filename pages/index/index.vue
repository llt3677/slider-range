<template>
  <view class="content">
    {{ rangeValue }}
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
		padding: 60rpx 16rpx;
	}

</style>
