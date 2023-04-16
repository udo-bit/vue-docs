<a name="cbZlZ"></a>
## 目标
完成多页签功能的测试
<a name="Mhdqh"></a>
## 开发
首先我们现在layouts目录下创建一个multi-tab的目录，然后我们再创建一个index.vue作为我们入口文件。<br />然后我们这一部分先写一些测试代码。
```vue
<script lang="ts" setup>

</script>

<template>
  <div class="flex h-48px bg-white w-100%" />
</template>

<style scoped>

</style>
```
接下来我们就来测试一下。<br />在base-layout/index.vue中导入
```vue
<script lang="ts" setup>
  import MultiTab from '../multi-tab/index.vue'
</script>
<template>
  <MixLayout
      v-if="layout.layout === 'mix'"
      v-model:collapsed="layout.collapsed"
      :logo="layout.logo"
      :title="layout.title"
      :active="active"
      :options="menuOptions"
      :show-sider-trigger="layout.showSiderTrigger"
      :sider-width="layout.siderWidth"
      :sider-collapsed-width="layout.siderCollapsedWidth"
    >
      <template #headerRight>
        <RightContent />
      </template>
      <MultiTab />
    </MixLayout>
</template>
```
接下来我们测试一下，会发现我们的样式是有问题的，他会存在一个边距，这就是我们之前写了一个内边距导致的，所以在这里我们需要删除这个内边距。<br />在mix-layout中删除给LayouContent传入的边距。<br />然后测试。我们的多页签的测试功能就测试完成了。
