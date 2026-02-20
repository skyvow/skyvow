# Taro React 使用原生组件的 Slot（插槽）

在 Taro React 中使用微信小程序原生组件的 slot，需要特殊处理。以下是完整的使用指南。

---

## 1. 引入原生组件

首先在页面或组件的配置文件中声明要使用的原生组件：

```json
// page.config.js 或 page.config.ts
export default definePageConfig({
  usingComponents: {
    "my-native-comp": "../../components/my-native-comp/index"
  }
})
```

## 2. 默认 Slot（单个插槽）

原生组件只有一个默认 slot 时，直接在标签内放入子元素即可：

**原生组件定义 (WXML)：**

```xml
<!-- components/my-native-comp/index.wxml -->
<view class="wrapper">
  <view class="header">原生组件头部</view>
  <slot></slot>
  <view class="footer">原生组件底部</view>
</view>
```

**Taro React 中使用：**

```tsx
import { View, Text } from '@tarojs/components'

export default function MyPage() {
  return (
    <View>
      <my-native-comp>
        <View>这是默认 slot 的内容</View>
      </my-native-comp>
    </View>
  )
}
```

## 3. 具名 Slot（多个插槽）⭐

当原生组件使用了**多个具名 slot** 时，这是最常见的需求场景。

### 3.1 原生组件开启多 slot 支持

```js
// components/my-native-comp/index.js
Component({
  options: {
    multipleSlots: true  // 必须开启
  },
  // ...
})
```

```xml
<!-- components/my-native-comp/index.wxml -->
<view class="wrapper">
  <slot name="header"></slot>
  <view class="content">组件内容</view>
  <slot name="footer"></slot>
</view>
```

### 3.2 Taro React 中使用具名 Slot

在 Taro React 中，通过给子元素添加 `slot` 属性来指定具名插槽：

```tsx
import { View, Text } from '@tarojs/components'

export default function MyPage() {
  return (
    <View>
      <my-native-comp>
        <View slot="header">
          <Text>我是 header 插槽内容</Text>
        </View>
        <View slot="footer">
          <Text>我是 footer 插槽内容</Text>
        </View>
      </my-native-comp>
    </View>
  )
}
```

> **关键点：** 使用 `slot="插槽名"` 属性来对应原生组件中 `<slot name="插槽名">` 的位置。

## 4. 实际案例：使用 vant-weapp 组件

以 `vant-weapp` 的 `van-cell` 组件为例，它提供了多个具名 slot：

### 4.1 配置原生组件路径

```json
// page.config.ts
export default definePageConfig({
  usingComponents: {
    "van-cell": "../../wxcomponents/vant/cell/index",
    "van-icon": "../../wxcomponents/vant/icon/index"
  }
})
```

### 4.2 使用具名 Slot

```tsx
import { View, Text, Image } from '@tarojs/components'

export default function MyPage() {
  return (
    <View>
      {/* van-cell 的 icon slot */}
      <van-cell title="单元格" value="内容">
        <van-icon slot="icon" name="star" />
      </van-cell>

      {/* van-cell 的 title slot 和 right-icon slot */}
      <van-cell>
        <View slot="title">
          <Text>自定义标题</Text>
          <Text style={{ color: '#999', fontSize: '12px' }}>描述信息</Text>
        </View>
        <van-icon slot="right-icon" name="arrow" />
      </van-cell>
    </View>
  )
}
```

## 5. 使用 NativeSlot 组件（Taro 3.5+）

从 **Taro 3.5** 版本开始，提供了 `NativeSlot` 组件用于渲染原生 slot，主要用于**自定义 Taro 组件作为原生组件的子组件时**传递 slot：

```tsx
import { View } from '@tarojs/components'
import { NativeSlot } from '@tarojs/components/lib/react'

// 自定义的 Taro 组件，内部需要渲染原生 slot
function MyWrapper({ children }) {
  return (
    <View className="my-wrapper">
      <my-native-comp>
        {children}
      </my-native-comp>
    </View>
  )
}

// 使用
export default function MyPage() {
  return (
    <MyWrapper>
      <View slot="header">Header 内容</View>
      <View slot="footer">Footer 内容</View>
    </MyWrapper>
  )
}
```

## 6. 注意事项

### 6.1 TypeScript 类型声明

在 TypeScript 中使用原生组件时，需要添加类型声明以消除类型错误：

```ts
// types/global.d.ts
declare namespace JSX {
  interface IntrinsicElements {
    'my-native-comp': React.DetailedHTMLProps<
      React.HTMLAttributes<HTMLElement> & {
        title?: string
        value?: string
        // 其他属性...
      },
      HTMLElement
    >
    'van-cell': React.DetailedHTMLProps<
      React.HTMLAttributes<HTMLElement> & {
        title?: string
        value?: string
        label?: string
        icon?: string
        border?: boolean
      },
      HTMLElement
    >
    'van-icon': React.DetailedHTMLProps<
      React.HTMLAttributes<HTMLElement> & {
        name?: string
        size?: string | number
        color?: string
        slot?: string
      },
      HTMLElement
    >
  }
}
```

### 6.2 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| slot 内容不显示 | 原生组件未开启 `multipleSlots` | 在组件的 `options` 中设置 `multipleSlots: true` |
| 编译报错找不到组件 | 路径配置错误 | 检查 `usingComponents` 中的路径是否正确 |
| TS 类型报错 | 缺少类型声明 | 添加 `global.d.ts` 类型声明文件 |
| slot 内容位置错乱 | slot 名称不匹配 | 确保 `slot="name"` 与原生组件中 `<slot name="">` 一致 |
| H5 端不生效 | 原生组件仅支持小程序 | H5 端需要用 Taro 组件实现同等功能，或使用跨端兼容方案 |

### 6.3 跨端兼容

原生组件的 slot 仅在小程序端生效。如果项目需要多端运行，建议通过 `process.env.TARO_ENV` 做条件编译：

```tsx
import { View, Text } from '@tarojs/components'

export default function MyPage() {
  return (
    <View>
      {process.env.TARO_ENV === 'weapp' ? (
        <van-cell title="单元格">
          <van-icon slot="right-icon" name="arrow" />
        </van-cell>
      ) : (
        <View className="cell">
          <Text>单元格</Text>
          <Text className="arrow">→</Text>
        </View>
      )}
    </View>
  )
}
```

## 7. 总结

```
                    ┌─────────────────────────────────┐
                    │  Taro React 使用原生组件 Slot     │
                    └────────────┬────────────────────┘
                                 │
                 ┌───────────────┼───────────────┐
                 ▼               ▼               ▼
          ┌──────────┐   ┌──────────────┐  ┌───────────┐
          │ 默认 Slot │   │  具名 Slot   │  │ NativeSlot│
          │          │   │              │  │ (3.5+)    │
          └──────────┘   └──────────────┘  └───────────┘
                │               │               │
                ▼               ▼               ▼
          直接放子元素     slot="name"     组件透传 slot
```

- **默认 slot**：直接在原生组件标签内放入子元素
- **具名 slot**：通过 `slot="name"` 属性指定插槽名称
- **NativeSlot（Taro 3.5+）**：用于 Taro 自定义组件透传 slot
- **TypeScript**：需添加全局类型声明
- **跨端**：使用 `process.env.TARO_ENV` 条件编译
