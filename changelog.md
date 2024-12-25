# 更新日志

## [1.0.0] - 2024-03-21

### 新增
- 双滑块范围选择器基础功能
- 支持设置最小值(min)和最大值(max)
- 支持自定义步长(step)设置
- 支持禁用状态(disabled)
- 自定义样式功能:
  - 滑动条背景色(backgroundColor)
  - 选中范围颜色(activeColor) 
  - 滑块大小(blockSize)
  - 滑块颜色(blockColor)
- 值格式化功能(format)
- 支持移动端触摸操作
- v-model双向绑定支持

### 事件
- update:modelValue - 值更新时触发
- change - 值变化时触发

### 技术细节
- 使用节流优化拖动性能
- 适配移动端触摸区域
- 支持精确的数值计算和范围控制 