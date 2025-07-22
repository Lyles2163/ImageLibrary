# OpenDataV低代码平台快速开发文档

## 新增组件流程

### 1.进入组件库目录下

所有的可拖拽组件都存放在`src/resource/components`目录下

```
cd src/resource/components
```

### 2.新建自定义组件组或子组件

1.新建自定义组件组（可以直接到已有的组件组创建子组件）（例如）

```
mkdir src/resource/components/echarts/PieChart/PlainPieChart
```

2.用npm初始化（其他默认， keywords: OpenDataV  author: yourname  license: (ISC) Apache-2.0）

```
npm init
```

![初始化](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/npm.png)

3.新增 index.ts,config.ts,type.ts文件

1.index.ts文件（例如）

```
import PlainPieChartComponent, { componentName } from './config'

// 默认导出对象，包含组件名称、异步加载的 Vue 组件、以及配置类
export default {
  componentName, // 组件名称（字符串）
  component: () => import('./PlainPieChart.vue'), // 异步导入 Vue 组件
  config: PlainPieChartComponent // 组件的配置类（包含元数据和可配置项）
}
```



2.config.ts文件（例如）

```
import { ComponentGroup, FormType } from '@/enum'
import type { PropsType } from '@/types/component'
import { BaseComponent } from '@/resource/models'
import { DataIntegrationMode } from '@/resource/models/data'
import { h, ref, watch } from 'vue'

// 组件名称常量
export const componentName = 'PlainPieChart'

// PlainPieChartComponent 继承自 BaseComponent，描述了饼图组件的元数据和可配置属性
class PlainPieChartComponent extends BaseComponent {
  // 响应式数据示例（可用于数据联动或预览）
  public reactiveData = ref(this.exampleData)

  // 构造函数，初始化组件的基本信息
  constructor(id?: string, name?: string, icon?: string) {
    super({
      component: componentName, // 组件唯一标识
      group: ComponentGroup.PIE, // 组件分组（饼图）
      name: name ? name : '普通饼状图', // 组件显示名称
      id, // 组件唯一ID
      width: 400, // 默认宽度
      height: 300, // 默认高度
      icon, // 组件图标
      dataIntegrationMode: DataIntegrationMode.UNIVERSAL // 数据集成模式
    })
  }


export default PlainPieChartComponent
```



3.PlainPieChart.vue文件

作用：

这是一个基于 ECharts 的饼状图 Vue 组件。它会根据外部传入的配置参数（如标题、半径、颜色等）动态渲染饼图，并且可以随着父容器的拖拽实时自适应大小。

主要功能：

- 通过 useProp 获取外部传入的配置参数。

- 通过 useEchart 封装 ECharts 的初始化、更新和自适应大小逻辑。

- 使用 v-resize 指令监听容器尺寸变化，自动调用 ECharts 的 resize 方法。

- 响应式地根据参数变化实时更新图表。

```
<template>
  <!-- 使用 ref 属性来获取 DOM 元素，v-resize 指令用于监听窗口大小变化 -->
  <div ref="chartDom" v-resize="resizeHandler" style="width: 100%; height: 100%"></div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import { random } from 'lodash-es'
import { useProp } from '@/resource/hooks'
import { useEchart } from '../../hooks'
import { PlainPieChartType } from '@/resource/components/echarts/PieChart/PlainPieChart/type'
import type PlainPieChartComponent from '@/resource/components/echarts/PieChart/PlainPieChart/config'

// 定义组件接收的属性
const props = defineProps<{
  component: PlainPieChartComponent
}>()
// 用于获取图表 DOM 元素的引用
const chartDom = ref<HTMLElement | null>(null)
// 解析并获取属性值
const { propValue } = useProp<PlainPieChartType>(props.component)
// @ts-ignore
// 这是一个组件，用于展示一个饼状图并随拖拽来适应大小
const { updateEchart, resizeHandler } = useEchart(chartDom)

// 生成 Echart 配置项
const getOption = () => ({
  title: {
    text: propValue.title.text,// 标题文本
    subtext: propValue.title.subText,// 副标题文本
    left: propValue.title.position,// 标题位置
    textStyle: {
      color: propValue.font.color,// 字体颜色
      fontSize: propValue.title.textFontSize // 标题字体大小
    }
  },
  tooltip: { trigger: 'item' },
  legend: { orient: 'vertical', left: 'left' },
  series: [
    {
      textStyle: {
        color: propValue.font.color,
        fontSize: propValue.font.fontSize
      },
      name: 'Access From',
      type: 'pie',
      radius: propValue.radius.size + '%',
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
          shadowColor: 'rgba(0, 0, 0, 0.5)'
        }
      }
    }
  ]
})

// 组件挂载完成后，初始化图表
onMounted(() => {
  updateEchart(getOption())
})

// 监听propValue变化, 并更新图表
watch(
  () => propValue,
  () => {
    updateEchart(getOption())
  },
  { deep: true }
)
</script>

```

### 新增组件示例

现在已经新增了一个新的组件

![新增组件](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/component.gif)





## 新增自定义属性编辑

### 1.新增Type.ts文件

作用：

定义了 PlainPieChart 组件所需的参数类型（TypeScript 接口），包括标题、半径、字体等所有可配置项的类型约束。（即自定义属性）这样可以保证传入参数的正确性和类型安全。

```
// PositionType 用于限定标题位置的类型，可以是字符串（'center'/'left'/'right'），也可以是数值或数组
type PositionType = 'center' | 'left' | 'right' | number | number[]

// PlainPieChartType 定义了 PlainPieChart 组件所有可配置参数的类型
export interface PlainPieChartType {
  // 饼图半径相关配置
  radius: {
    size: number // 饼图半径百分比（如 60 表示 60%）
  }
  // 标题相关配置
  title: {
    text: string // 主标题文本
    subText: string // 副标题文本
    textFontSize: number // 主标题字体大小
    subTextFontSize: number // 副标题字体大小
    position: PositionType // 标题位置
  }
  // 字体相关配置
  font: {
    color: string // 字体颜色
    fontSize: number // 字体大小
  }
}
```

2.config.ts文件

```
 // 组件可配置属性列表（用于生成配置面板）
  _prop: PropsType[] = [
    {
      label: '圆角设置', // 配置分组标题
      prop: 'radius',   // 对应 propValue.radius
      children: [
        {
          prop: 'size', // 对应 propValue.radius.size
          label: '圆角大小',
          type: FormType.NUMBER,
          componentOptions: {
            defaultValue: 60 //默认值
          },
        },
      ]
    },
    {
      label: '标题设置',
      prop: 'title',
      children: [
        {
          prop: 'text',
          label: '标题',
          type: FormType.TEXT,
          componentOptions: {
            defaultValue: '标题'
          },
        },
        {
          prop: 'subText',
          label: '副标题',
          type: FormType.TEXT,
          componentOptions: {
            defaultValue: '副标题'
          },
        },
        {
          prop: 'textFontSize',
          label: '标题字体大小',
          type: FormType.NUMBER,
          componentOptions: {
            defaultValue: 30,
            suffix: () => h('span', {}, 'vh')
          }
        },
        {
          prop: 'position',
          label: '标题位置',
          type: FormType.SELECT,
          componentOptions: {
            defaultValue: 'center',
            options: [
              { value: 'center', label: '居中' },
              { value: 'left', label: '靠左' },
              { value: 'right', label: '靠右' }
            ]
          }
        },
      ]
    },
  ]
```



现在就可以使用自定义组件了

![新增属性](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/Attributes.gif)

## 其他文件说明：

#### 1.enum/index.ts文件

(全局通用的“枚举类型和常量定义文件)

作用：

统一管理和规范项目中用到的各种“类型、分组、表单类型、图表类型、常量”等



![组件图标](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/icon.png)![新增分组信息](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/enum.png)

#### 2.hook.ts文件

作用：是一个ECharts 图表的通用 Hook 工具，它的作用是封装 ECharts 的初始化、主题、销毁、更新和自适应大小的逻辑

. 统一 ECharts 实例的生命周期管理
自动在组件挂载时初始化 ECharts 实例，并注册自定义主题（如 mydark）。
自动在组件卸载时清理和销毁 ECharts 实例，防止内存泄漏。

2. 提供图表自适应大小的能力
返回的 resizeHandler 方法可以配合 v-resize 指令使用，当图表容器尺寸变化时，自动调用 ECharts 的 resize 方法，保证图表始终自适应父容器。
3. 提供安全的图表更新方法
返回的 updateEchart 方法可以安全地更新 ECharts 的配置（option），并自动处理异常。
4. 主题统一
通过 echarts.registerTheme('mydark', mydark)，让所有用这个 Hook 的图表都能统一使用自定义主题。

![组件图标](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/changeSize.gif)

#### 3.userPorp/index文件

作用：

1. 把外部传进来的组件配置（props.component）变成响应式对象，叫做 propValue。

2. ```
   const { propValue } = useProp<PlainPieChartType>(props.component)
   ```

3. propValue 是响应式的，意思是：如果外部配置发生变化（比如用户在界面上修改了标题、颜色等），propValue 里的内容会自动更新，组件会自动用新值渲染。

1. 类型安全，因为用了 <PlainPieChartType>，你在写 propValue.xxx 时会有智能提示和类型检查，减少出错。

1. 用法简单，你可以直接用 propValue.title.text、propValue.radius.size 等，像用普通对象一样获取配置参数，非常方便。



![属性流程](https://github.com/Lyles2163/ImageLibrary/raw/main/OpenDataV/AttributeProcess.png)
