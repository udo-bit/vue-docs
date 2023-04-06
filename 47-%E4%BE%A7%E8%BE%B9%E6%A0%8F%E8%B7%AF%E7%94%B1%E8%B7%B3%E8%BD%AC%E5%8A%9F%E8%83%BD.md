<a name="U56AC"></a>
## 目标
完成侧边栏路由跳转功能开发
<a name="Zc9bJ"></a>
## 开发
上节课我们让菜单展示出来了，但是我们现在还没有办法点击侧边栏进行路由跳转，那么我们现在来一起实现一下这一部分的功能。<br />首先我们看一下再naive-ui中如何实现这个功能[menu router-link](https://www.naiveui.com/zh-CN/os-theme/components/menu#router-link.vue)<br />接下来为了方便我们实现这个功能，我们还是将这一部分拆解成一个组件的方式去实现：<br />在layouts/side-menu中创建一个side-title.vue的文件
```vue
<script lang="ts" setup>
import type { RouteRecordRaw } from 'vue-router'

const props = defineProps<{ route: RouteRecordRaw }>()
const path = computed(() => props.route.path)
const title = computed(() => props.route?.meta?.title)
const hasChildren = computed(() => props?.route?.children && props.route.children?.length > 0)
</script>

<template>
  <template v-if="hasChildren">
    {{ title }}
  </template>
  <template v-else>
    <router-link :to="path">
      {{ title }}
    </router-link>
  </template>
</template>

```
然后在generate-menu.ts中使用：
```typescript
import SideTitle from '~/layouts/side-menu/side-title.vue'

const currentMenu: MenuOption = {
  key: route.path,
  label: () => h(SideTitle, { route }),
}
```
最终我们来看一下实现的效果。
<a name="GrSO5"></a>
## 拓展
我们既然通过这种方式实现了路由跳转，我们是不是也可以实现跳转全链接的方案呢？<br />接下来我们来实现一下这一部分，<br />在side-title.vue中:
```vue
<script lang="ts" setup>
const isFullPath = computed(() => path.value.startsWith('http'))
const target = computed(() => props?.route?.meta?.target ?? '_blank')
</script>
<template>
<!--   ... -->
  <template v-else-if="isFullPath">
    <a :href="path" :target="target">{{ title }}</a>
  </template>
<!--   ... -->
</template>
```
最终我们来测试一下。<br />大家还可以拓展一些其他的用法，这里看大家拓展自己思维。
