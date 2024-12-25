# SliderRange 双滑块范围选择器

一个功能强大的双滑块范围选择器组件,可用于价格区间、数值范围等场景的选择。支持自定义样式、步长设置,具有良好的移动端适配性和交互体验。

## 特性

- 支持设置最小值和最大值范围
- 可自定义滑块样式
- 支持步长设置
- 实时数值反馈
- 适配移动端触摸操作

## 安装

将 `llt-slider-range` 组件复制到你的项目中的。

## 使用方法

## 属性说明

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| modelValue | Array | [0, 100] | 当前选中的范围值 |
| min | Number | 0 | 最小值 |
| max | Number | 100 | 最大值 |
| step | Number | 1 | 步长 |
| disabled | Boolean | false | 是否禁用 |
| backgroundColor | String | '#F6F6F6' | 滑动条背景色 |
| activeColor | String | '#4DB8F6' | 选中范围的颜色 |
| blockSize | Number | 48 | 滑块大小(rpx) |
| blockColor | String | '#fff' | 滑块颜色 |
| format | Function | val => val | 数值格式化函数 |


## 事件说明

| 事件名 | 说明 | 回调参数 |
|--------|------|----------|
| update:modelValue | 值更新时触发 | (value: number[]) 当前选择的范围值 |
| change | 值变化时触发 | (value: number[]) 当前选择的范围值 |

## 示例

### 基础用法
```vue
<template>
  <llt-slide-range v-model="rangeValue" />
</template>
<script>
export default {
  data() {
    return {
      rangeValue: [20, 80] // 设置初始范围值
    }
  }
}
</script>
```

### 自定义样式

```vue
<template>
  <llt-slide-range
    v-model="rangeValue"
    :min="0"
    :max="1000"
    :step="10"
    active-color="#FF6B6B"
    background-color="#E9ECEF"
    :block-size="60"
    block-color="#FF6B6B"
  />
</template>
```

### 格式化显示值

```vue
<template>
  <llt-slide-range
    v-model="rangeValue"
    :format="val => `${val}°C`"
  />
</template>
```
# 注意事项

1. value 数组的两个值必须在 min 和 max 之间
2. 确保 step 的值大于 0
3. 在移动端使用时建议适当增大滑块的触摸区域

## License MIT