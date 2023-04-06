<a name="Eg9Jr"></a>
## 目标
完成侧边栏路由图标的开发和展示。
<a name="hAhcd"></a>
## 开发
这节课我们来完成一下侧边栏图标的渲染，我们还是看一下naive-ui是怎么实现的。<br />然后我们还是在side-menu下创建一个side-icon.vue
```vue
<script lang="ts" setup>
import * as icons from '@vicons/antd'

const props = defineProps<{
  icon: string
}>()

const comp = computed(() => (icons as any)[props.icon])
</script>

<template>
  <n-icon :component="comp" />
</template>

```

这里我采用的是全量引入所有与antd相关的图标，你也可以按需引入，在side-menu文件夹下创建一个icons.ts的文件，如下：
```typescript
import { HddFilled, HddOutlined, HddTwotone } from '@vicons/antd'

export default {
  HddFilled,
  HddOutlined,
  HddTwotone,
}

```
然后修改引入：
```vue
<script lang="ts" setup>
// import * as icons from '@vicons/antd'
import icons from './icons'

</script>
```

然后我们在generate-menu.ts的文件
```typescript
import SideIcon from '~/layouts/side-menu/side-icon.vue'

if (route?.meta?.icon)
    currentMenu.icon = () => h(SideIcon, { icon: route?.meta?.icon })
```
我们来看一下整个的实现效果。
