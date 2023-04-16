<a name="mfW7y"></a>
## 目标
完成基础的多页签功能开发

<a name="pTW7L"></a>
## 开发
开发多页签我们用到的naive-ui的组件就是tabs，那么我们直接在官方拷贝一份拿过来试一下。
```vue
<template>
  <n-tabs
    v-model:value="name"
    type="card"
    closable
    tab-style="min-width: 80px;"
    @close="handleClose"
  >
    <n-tab-pane
      v-for="panel in panels"
      :key="panel"
      :tab="panel.toString()"
      :name="panel"
    >
      {{ panel }}
    </n-tab-pane>
  </n-tabs>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'
import { useMessage } from 'naive-ui'

export default defineComponent({
  setup () {
    const nameRef = ref(1)
    const message = useMessage()
    const panelsRef = ref([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15])
    function handleClose (name: number) {
      const { value: panels } = panelsRef
      if (panels.length === 1) {
        message.error('最后一个了')
        return
      }
      message.info('关掉 ' + name)
      const index = panels.findIndex((v) => name === v)
      panels.splice(index, 1)
      if (nameRef.value === name) {
        nameRef.value = panels[index]
      }
    }
    return {
      panels: panelsRef,
      name: nameRef,
      handleClose
    }
  }
})
</script>
```
拷贝过来之后我们来看一下现在的效果。<br />大家会看到我们左侧紧挨着我们的侧边栏和我们顶部，我感觉这样不太好看，我们让他在边上有一些边距会好看一些。<br />如下：
```vue
<template>
  <n-tabs
    v-model:value="nameRef"
    type="card"
    closable
    class="pt-6px bg-white dark:bg-transparent"
    tab-style="min-width: 80px;"
    @close="handleClose"
  >
<!--     里面我们直接通过前置插槽的方式进行处理，因为我们给全局padding下面的边框也会跟着有一个padding -->
     <template #prefix>
      <div class="pl-16px" />
    </template>
</n-tabs>
```

我们来测试一下暗黑模式和亮色模式，是否有问题，我们发现我们在亮色模式的时候，模糊不清，默认他是继承主色的，所以我们这里需要进行一下处理，当是亮色模式的时候背景色是白色，暗色模式的时候背景色我们设置为透明主题。<br />最后我们把代码转化成setup的形式：
```vue
<script lang="ts" setup>
const nameRef = ref<number>(1)
const message = useMessage()
const panels = $ref([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15])
function handleClose(name: number) {
  if (panels.length === 1) {
    message.error('最后一个了')
    return
  }
  message.info(`关掉 ${name}`)
  const index = panels.findIndex(v => name === v)
  panels.splice(index, 1)
  if (nameRef.value === name)
    nameRef.value = panels[index]
}
</script>

<template>
  <n-tabs
    v-model:value="nameRef"
    type="card"
    closable
    class="pt-6px bg-white dark:bg-transparent"
    tab-style="min-width: 80px;"
    @close="handleClose"
  >
    <template #prefix>
      <div class="pl-12px" />
    </template>
    <n-tab-pane
      v-for="panel in panels"
      :key="panel"
      :tab="panel.toString()"
      :name="panel"
    />
  </n-tabs>
</template>
```
然后测试。<br />最后我们再优化一下内容区域，要有一定的边距。
```vue
<script lang="ts" setup>
import MultiTab from '../multi-tab/index.vue'
</script>

<template>
  <MultiTab />
  <main class="m-24px">
    <router-view />
  </main>
</template>

```
然后测试