# SliderRange 双滑块范围选择器

> 一个轻量级但功能强大的双滑块范围选择器组件，完美适配移动端和PC端，为您的项目提供直观的范围选择体验。

## 🌟 特性

- ✨ 灵活的范围设置 - 支持自定义最大最小值
- 📏 精确的步长控制 - 可设置数值变化的最小单位
- 🎨 丰富的样式定制 - 支持自定义滑块、轨道样式
- 📱 完美的跨端适配 - 支持H5/App/小程序多端使用
- 🔄 双向绑定支持 - 支持v-model语法
- 🎯 刻度线显示 - 直观的数值参考
- ⌨️ 键盘操作支持 - 提升无障碍体验
- 🚫 禁用状态 - 支持只读模式

## 📦 安装

### 下载使用
将 `llt-slider-range` 组件复制到你的项目中即可。

## 🚀 快速开始

### H5/App使用示例
```vue
<template>
  <llt-slide-range v-model="rangeValue" />
</template>

<script>
export default {
  data() {
    return {
      rangeValue: [30, 50] // 设置初始范围值
    }
  }
}
</script>
```

### 微信小程序使用示例
```vue
<template>
  <llt-slide-range 
    :model-value="rangeValue" 
    @change="handleChange"
  />
</template>

<script>
export default {
  data() {
    return {
      rangeValue: [30, 50]
    }
  },
  methods: {
    handleChange(val) {
      this.rangeValue = val
    }
  }
}
</script>
```

## 📝 API文档

### Props 属性

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| modelValue/v-model | Array | [0, 100] | 当前选中的范围值 |
| min | Number | 0 | 最小可选值 |
| max | Number | 100 | 最大可选值 |
| step | Number | 1 | 步长，必须大于0 |
| disabled | Boolean | false | 是否禁用 |
| backgroundColor | String | '#F6F6F6' | 滑动条背景色 |
| activeColor | String | '#4DB8F6' | 选中范围的颜色 |
| blockSize | Number | 48 | 滑块大小(rpx) |
| blockColor | String | '#fff' | 滑块颜色 |
| format | Function | val => val | 数值格式化函数 |

### Events 事件

| 事件名 | 说明 | 回调参数 |
|--------|------|----------|
| update:modelValue | 值更新时触发 | (value: number[]) |
| change | 值变化时触发 | (value: number[]) |

## 💡 高级用法

### 价格范围选择器
```vue
<template>
  <llt-slide-range
    v-model="priceRange"
    :min="0"
    :max="10000"
    :step="100"
    :format="formatPrice"
    active-color="#FF6B6B"
  />
</template>

<script>
export default {
  data() {
    return {
      priceRange: [1000, 5000]
    }
  },
  methods: {
    formatPrice(val) {
      return `￥${val}`
    }
  }
}
</script>
```

### 温度选择器
```vue
<template>
  <llt-slide-range
    v-model="tempRange"
    :min="-20"
    :max="40"
    :format="val => `${val}°C`"
    active-color="#2196F3"
  />
</template>
```

## ⚠️ 注意事项

1. 确保传入的范围值在 min 和 max 之间
2. step 值必须为正数
3. 移动端使用时建议适当增大 blockSize 以提升触控体验
4. 小程序端需使用 model-value + change 事件方式实现双向绑定
5. 建议在父容器设置合适的宽度，以获得最佳显示效果

## 🔗 相关链接

- [GitHub 仓库](https://github.com/llt3677/slider-range)
- [插件市场](https://ext.dcloud.net.cn/plugin?id=21575)
- [问题反馈](https://github.com/llt3677/slider-range/issues)

## 📄 License 

[MIT](https://opensource.org/licenses/MIT)

---

如果这个组件对你有帮助，欢迎 star ⭐️ 支持一下！
