# 🚀 OpenDataV 低代码平台快速开发文档

> ✨ 快速构建可视化组件｜基于 Vue3 + ECharts + TypeScript  
> 🔗 官方仓库：[yu-bruce/openDataV: OpenDataV 是一个纯前端的拖拽式、可视化、低代码数据可视化🌈开发平台，你可以用它自由的拼接成各种✨炫酷的大屏，同时支持用户方便的开发自己的组件并接入平台。 (github.com)](https://github.com/yu-bruce/openDataV)

📌 示例：![演示](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/test.gif)

---

## 🧩 新增组件流程

### 1️⃣ 进入组件库目录

所有可拖拽组件均存放在 `src/resource/components` 目录下：

```bash
cd src/resource/components
```

📁 **路径结构建议**：

```
components/
└── echarts/
    └── PieChart/
        └── PlainPieChart/     ← 新增组件目录
            ├── index.ts
            ├── config.ts
            ├── type.ts
            └── PlainPieChart.vue
```

---

### 2️⃣ 创建自定义组件

#### ✅ 步骤一：创建组件目录（以饼图为例）

```bash
mkdir -p src/resource/components/echarts/PieChart/PlainPieChart
cd src/resource/components/echarts/PieChart/PlainPieChart
```

> 💡 提示：你也可以在已有分组中新增子组件，保持分类清晰。

---

#### ✅ 步骤二：初始化 npm 包

```bash
npm init
```

🔧 建议填写如下信息：

| 字段       | 推荐值                                    |
| ---------- | ----------------------------------------- |
| `name`     | `@opendatav/plain-pie-chart`              |
| `keywords` | `OpenDataV`, `visualization`, `component` |
| `author`   | 你的名字                                  |
| `license`  | `Apache-2.0`                              |

📌 示例截图：  
![npm init](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/npm.png)

---

#### ✅ 步骤三：创建核心文件

##### 📄 `index.ts` —— 组件入口导出

```ts
import PlainPieChartComponent, { componentName } from './config'

export default {
  componentName,
  component: () => import('./PlainPieChart.vue'),
  config: PlainPieChartComponent
}
```

🟢 **作用**：统一暴露组件元数据和异步加载逻辑。

---

##### 📄 `config.ts` —— 组件配置类

```ts
import { ComponentGroup, FormType } from '@/enum'
import type { PropsType } from '@/types/component'
import { BaseComponent } from '@/resource/models'
import { DataIntegrationMode } from '@/resource/models/data'
import { h, ref, watch } from 'vue'

export const componentName = 'PlainPieChart'

class PlainPieChartComponent extends BaseComponent {
  public reactiveData = ref(this.exampleData)

  constructor(id?: string, name?: string, icon?: string) {
    super({
      component: componentName,
      group: ComponentGroup.PIE,
      name: name ? name : '普通饼状图',
      id,
      width: 400,
      height: 300,
      icon,
      dataIntegrationMode: DataIntegrationMode.UNIVERSAL
    })
  }

  // 自定义属性配置面板
  _prop: PropsType[] = [
    {
      label: '圆角设置',
      prop: 'radius',
      children: [
        {
          prop: 'size',
          label: '圆角大小',
          type: FormType.NUMBER,
          componentOptions: { defaultValue: 60 }
        }
      ]
    },
    {
      label: '标题设置',
      prop: 'title',
      children: [
        {
          prop: 'text',
          label: '主标题',
          type: FormType.TEXT,
          componentOptions: { defaultValue: '标题' }
        },
        {
          prop: 'subText',
          label: '副标题',
          type: FormType.TEXT,
          componentOptions: { defaultValue: '副标题' }
        },
        {
          prop: 'textFontSize',
          label: '字体大小',
          type: FormType.NUMBER,
          componentOptions: {
            defaultValue: 30,
            suffix: () => h('span', {}, 'vh')
          }
        },
        {
          prop: 'position',
          label: '位置',
          type: FormType.SELECT,
          componentOptions: {
            defaultValue: 'center',
            options: [
              { value: 'center', label: '居中' },
              { value: 'left', label: '靠左' },
              { value: 'right', label: '靠右' }
            ]
          }
        }
      ]
    }
  ]
}

export default PlainPieChartComponent
```

🛠️ **功能说明**：

- 定义组件名称、分组、默认尺寸等元信息。
- 使用 `_prop` 构建可视化配置表单。
- 支持响应式预览与联动。

---

##### 📄 `type.ts` —— 类型定义（TypeScript 接口）

```ts
type PositionType = 'center' | 'left' | 'right' | number | number[]

export interface PlainPieChartType {
  radius: {
    size: number
  }
  title: {
    text: string
    subText: string
    textFontSize: number
    subTextFontSize: number
    position: PositionType
  }
  font: {
    color: string
    fontSize: number
  }
}
```

🧩 **优势**：

- ✅ 类型安全
- 💡 IDE 智能提示
- 🛠 易于维护和扩展

---

##### 📄 `PlainPieChart.vue` —— 图表渲染组件

```vue
<template>
  <div ref="chartDom" v-resize="resizeHandler" style="width: 100%; height: 100%"></div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import { random } from 'lodash-es'
import { useProp } from '@/resource/hooks'
import { useEchart } from '../../hooks'
import { PlainPieChartType } from './type'
import type PlainPieChartComponent from './config'

const props = defineProps<{ component: PlainPieChartComponent }>()
const chartDom = ref<HTMLElement | null>(null)
const { propValue } = useProp<PlainPieChartType>(props.component)
const { updateEchart, resizeHandler } = useEchart(chartDom)

const getOption = () => ({
  title: {
    text: propValue.title.text,
    subtext: propValue.title.subText,
    left: propValue.title.position,
    textStyle: {
      color: propValue.font.color,
      fontSize: propValue.title.textFontSize
    }
  },
  tooltip: { trigger: 'item' },
  legend: { orient: 'vertical', left: 'left' },
  series: [
    {
      name: 'Access From',
      type: 'pie',
      radius: `${propValue.radius.size}%`,
      data: [
        { value: random(1000, 5000), name: '李' },
        { value: random(1000, 5000), name: '张' },
        { value: random(1000, 5000), name: '赵' },
        { value: random(1000, 5000), name: '钱' },
        { value: random(1000, 5000), name: '孙' }
      ],
      emphasis: {
        itemStyle: {
          shadowBlur: 10,
          shadowOffsetX: 0,
          shadowColor: 'rgba(0,0,0,0.5)'
        }
      }
    }
  ]
})

onMounted(() => updateEchart(getOption()))
watch(() => propValue, () => updateEchart(getOption()), { deep: true })
</script>
```

🎯 **核心能力**：

- 🔄 响应式更新图表
- 🖼 自适应容器缩放（通过 `v-resize`）
- 🎨 动态绑定样式与数据
- ⚙️ 封装 ECharts 生命周期管理

✅ 效果演示：  
![新增组件效果](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/component.gif)

---

## 🎨 新增自定义属性编辑器

### 🧱 添加 `type.ts` 定义类型约束

如前所述，确保每个字段都有明确的类型定义，便于校验和提示。

### 🔧 在 `config.ts` 中配置 `_prop` 表单结构

系统将自动根据 `_prop` 生成右侧配置面板 👉

✨ 实时生效效果：  
![新增属性面板](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/Attributes.gif)

💡 **支持的表单控件类型**（来自 `FormType`）：

| 类型     | 控件       |
| -------- | ---------- |
| `TEXT`   | 文本输入框 |
| `NUMBER` | 数字输入框 |
| `COLOR`  | 颜色选择器 |
| `SELECT` | 下拉选择   |
| `SWITCH` | 开关切换   |
| `SLIDER` | 滑块调节   |

> 📌 所有类型定义详见：`@/enum/index.ts`

---

## 📘 其他关键文件说明

### 📂 `enum/index.ts` —— 全局枚举常量中心

📍 路径：`@/enum/index.ts`

🧾 主要内容包括：

```ts
export enum ComponentGroup {
  PIE = 'pie',
  LINE = 'line',
  BAR = 'bar'
}

export enum FormType {
  TEXT = 'text',
  NUMBER = 'number',
  COLOR = 'color',
  SELECT = 'select'
}
```

🎯 **用途**：

- 统一分组命名
- 规范配置项类型
- 避免魔法字符串

🖼️ 示例图标与分组展示：  
![组件分组与图标](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/icon.png)  
![枚举定义截图](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/enum.png)

---

### 🔌 `hooks/useEchart.ts` —— ECharts 通用 Hook

📦 封装了 ECharts 的常用操作：

```ts
const { updateEchart, resizeHandler } = useEchart(chartDom)
```

✅ 功能亮点：

| 功能         | 描述                         |
| ------------ | ---------------------------- |
| 🧱 初始化实例 | 自动创建 ECharts 实例        |
| 🎨 主题注册   | 支持 `mydark` 等主题统一风格 |
| 🔄 自动重绘   | 配合 `v-resize` 实现响应式   |
| 🧹 内存清理   | 卸载时自动销毁实例防泄漏     |

🎬 动态适配演示：  
![图表自适应大小](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/changeSize.gif)

---

### 🔄 `hooks/useProp.ts` —— 属性响应式转换器

```ts
const { propValue } = useProp<PlainPieChartType>(props.component)
```

🧠 工作原理流程图：  
![ ](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/AttributeProcess.png)

🔍 **三大优点**：

1. 🔄 外部配置变化 → `propValue` 自动更新
2. 💡 类型推导精准，IDE 友好
3. 🧩 使用简单：`propValue.title.text` 直接访问

---

## ✅ 总结：新增一个组件只需 5 步

| 步骤 | 操作                              | 图标 |
| ---- | --------------------------------- | ---- |
| 1    | 创建组件目录                      | 📁    |
| 2    | `npm init` 初始化                 | 🔧    |
| 3    | 编写 `type.ts` 定义参数类型       | 🧩    |
| 4    | 编写 `config.ts` 配置元信息与表单 | 🛠️    |
| 5    | 编写 `.vue` 文件实现视图逻辑      | 🖼    |

🎉 完成后即可在设计器中直接拖拽使用！

---

## 📣 加入我们

🤝 欢迎贡献组件！  
📖 文档持续更新中...  
💬 如有疑问，请提交 Issue 或联系作者。

---

> ✅ *让数据可视化更简单 —— OpenDataV*  
> 🌐 构建属于你的大屏应用，从一个组件开始！

---

📄 **版本**: v1.0.0  
📅 **最后更新**: 2025年7月22日

👤 Author: **李阳**  



