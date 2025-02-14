## 目标
1. 开发视图断点查询可组合式api
2. 基本的移动端布局

## 常用的视图断点枚举值
我们在开发中和视图大小相关的代名词像是：xl、lg、md等等。
我们这里定义的视图大小范围值也是一样的。
### 创建组合式api文件
在composables文件夹下创建一个query-breakpoints的文件。
然后首先定义我们的断点枚举值：
```typescript
export const breakpointsEnum = {
  xl: 1600,
  lg: 1199,
  md: 991,
  sm: 767,
  xs: 575,
}


```

接下来我们通过vueuse中的useBreakpoints 来实现我们响应式的值的功能。
在query-breakpoints中：
```typescript
export const useQueryBreakpoints = () => {
  const breakpoints = reactive(useBreakpoints(breakpointsEnum))

  const isMobile = breakpoints.smaller('sm')
  const isPad = breakpoints.between('sm', 'md')
  const isDesktop = breakpoints.greater('md')
  return { breakpoints, isMobile, isPad, isDesktop }
}
```
分别获取到我们的三种模式：
移动端布局、pad端、桌面端

## 创建移动端布局
接下来我们先创建基础的移动端的布局模式，在layouts中创建一个mobile-layout的布局文件夹，然后创建一个index.vue的文件。我们直接拷贝top-layout的布局进行改造。
```vue
<script lang="ts" setup>
import { LayoutContent, Logo, Title } from '~/layouts/common'
const props = withDefaults(defineProps<{
  headerHeight?: number
  logo?: string
  title?: string
  inverted?: boolean
}>(), {
  headerHeight: 48,
})

const headerHeightVar = computed(() => `${props.headerHeight}px`)
</script>

<template>
  <n-layout class="h-screen" style="--n-color:var(--pro-admin-layout-content-bg)">
    <n-layout-header
      :inverted="inverted"
      class="pro-admin-mix-layout-header flex justify-between items-center px-4"
    >
      <div class="flex items-center">
        <Logo :src="logo" />
        <Title :title="title" />
      </div>

      <slot name="headerRight">
        <div>
          右侧
        </div>
      </slot>
    </n-layout-header>
    <LayoutContent content-style="padding: 24px;">
      <slot />
    </LayoutContent>
  </n-layout>
</template>

<style scoped>
.pro-admin-mix-layout-header{
  height: v-bind(headerHeightVar);
}
</style>

```
当我们收起来的时候，我们不需要让标题再展示在header中，占用空间。
所以我们删掉先把title删除掉。
侧边栏我们使用抽屉代替。如下：
```vue
<script lang="ts" setup>
import { LayoutContent, Logo, Title } from '~/layouts/common'
const props = withDefaults(defineProps<{
  headerHeight?: number
  logo?: string
  title?: string
  inverted?: boolean
  visible?: boolean
}>(), {
  headerHeight: 48,
  visible: false,
})

defineEmits(['update:visible'])
const headerHeightVar = computed(() => `${props.headerHeight}px`)
</script>

<template>
  <n-layout class="h-screen" style="--n-color:var(--pro-admin-layout-content-bg)">
    <n-layout-header
      :inverted="inverted"
      class="pro-admin-mix-layout-header flex justify-between items-center px-4"
    >
      <div class="flex items-center">
        <Logo :src="logo" />
      </div>

      <slot name="headerRight">
        <div>
          右侧
        </div>
      </slot>
    </n-layout-header>
    <LayoutContent content-style="padding: 24px;">
      <slot />
    </LayoutContent>
  </n-layout>
  <n-drawer
    :show="visible"
    :width="240"
    placement="left"
    @update:show="(val) => $emit('update:visible', val)"
  >
    <n-drawer-content>
      《斯通纳》是美国作家约翰·威廉姆斯在 1965 年出版的小说。
    </n-drawer-content>
  </n-drawer>
</template>

<style scoped>
.pro-admin-mix-layout-header{
  height: v-bind(headerHeightVar);
}
</style>

```

接下来我们在base-layout中支持一下我们的移动端布局
```vue
<script lang="ts" setup>
import MixLayout from '../mix-layout/index.vue'
import SideLayout from '../side-layout/index.vue'
import TopLayout from '../top-layout/index.vue'
import MobileLayout from '../mobile-layout/index.vue'
const appStore = useAppStore()
const { layout } = storeToRefs(appStore)
const { isMobile } = useQueryBreakpoints()
</script>

<template>
  <MobileLayout
    v-if="isMobile"
    :logo="layout.logo"
    :title="layout.title"
  >
    <template #headerRight>
      <div>
        测试右侧插槽
      </div>
    </template>
    <router-view />
  </MobileLayout>
  <template v-else>
    <MixLayout
      v-if="layout.layout === 'mix'"
      v-model:collapsed="layout.collapsed"
      :logo="layout.logo"
      :title="layout.title"
      :show-sider-trigger="layout.showSiderTrigger"
      :sider-width="layout.siderWidth"
      :sider-collapsed-width="layout.siderCollapsedWidth"
    >
      <template #headerRight>
        <div>
          测试右侧插槽
        </div>
      </template>
      <router-view />
    </MixLayout>
    <SideLayout
      v-if="layout.layout === 'side'"
      v-model:collapsed="layout.collapsed"
      :logo="layout.logo"
      :title="layout.title"
      :show-sider-trigger="layout.showSiderTrigger"
      :sider-width="layout.siderWidth"
      :sider-collapsed-width="layout.siderCollapsedWidth"
    >
      <template #headerRight>
        <div>
          测试右侧插槽
        </div>
      </template>
      <router-view />
    </SideLayout>
    <TopLayout
      v-if="layout.layout === 'top'"
      v-model:collapsed="layout.collapsed"
      :logo="layout.logo"
      :title="layout.title"
      :show-sider-trigger="layout.showSiderTrigger"
      :sider-width="layout.siderWidth"
      :sider-collapsed-width="layout.siderCollapsedWidth"
    >
      <template #headerRight>
        <div>
          测试右侧插槽
        </div>
      </template>
      <router-view />
    </TopLayout>
  </template>
</template>
```
接下来我们实现一下展开收起部分
在stores/app.ts中做如下新增：
```typescript
import { layoutThemeConfig } from '~/config/layout-theme'
export const useAppStore = defineStore('app', () => {
  const defaultTheme = import.meta.env.DEV ? layoutThemeConfig : useLayoutTheme()
  const layout = reactive(unref(defaultTheme))
  const visible = ref(false)

  const updateVisible = (val: boolean) => {
    visible.value = val
  }

  const updateCollapsed = (val: boolean) => {
    layout.collapsed = val
  }

  return {
    layout,
    visible,
    updateVisible,
    updateCollapsed,
  }
})

```
然后再base-layout中导入：
```typescript
const { layout, visible } = storeToRefs(appStore)
```
在布局中使用：
```vue
<MobileLayout
    v-if="isMobile"
    v-model:visible="visible"
    :logo="layout.logo"
    :title="layout.title"
  >
    <template #headerRight>
      <div>
        测试右侧插槽
      </div>
    </template>
    <router-view />
</MobileLayout>
```

接下来我们在mobile-layout.vue中配置展开的图标：
我们的图标库使用官方给我们提供的库地址，
首先我们先来安装一下图标库，为了方便我们还是使用antd的图标库：
```shell
pnpm add -D @vicons/antd
```
然后在mobile-layout中使用：
```vue
<script lang="ts" setup>
import { MenuUnfoldOutlined } from '@vicons/antd'
const emit = defineEmits(['update:visible'])
const onShow = () => {
  emit('update:visible', true)
}
</script>
<template>
  <div class="flex items-center">
    <Logo :src="logo" size="26" />
    <n-icon size="20" class="ml-2" @click="onShow">
      <MenuUnfoldOutlined />
    </n-icon>
  </div>
</template>
```
测试实现效果
