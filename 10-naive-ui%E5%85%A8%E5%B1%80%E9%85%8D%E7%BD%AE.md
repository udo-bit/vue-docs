## 目标
1. 配置naive-ui的全局配置。
2. 测试各个弹层的全局配置是否可以正常使用。

## 全局配置
在我们开发前端中后台管理框架的时候，全局化配置是必不可少的。其中我们可以通过naive-ui的全局配置可以做很多事情，比如我们的多主题模式，暗黑模式，多语言等等。

### 在App.vue中
我们在app.vue中删除之前测试的代码直接导入如下代码：
```vue
<script setup lang="ts">
</script>

<template>
  <n-config-provider>
    <router-view />
  </n-config-provider>
</template>

```
## 各种弹层的注入
在naive-ui中，默认不会初始化全局的message、dialog、notification 以及loading-bar 。也是通过注入的方式注入进来，如果我们把所有的注入都放到app.vue中层级可能会很深，所以我们在components目录下创建文件一个app-provider.vue的文件来注册一下。
app-provider.vue
```vue

<template>
  <n-message-provider>
    <n-dialog-provider>
      <n-notification-provider>
        <n-loading-bar-provider>
          <slot />
        </n-loading-bar-provider>
      </n-notification-provider>
    </n-dialog-provider>
  </n-message-provider>
</template>

```

在app.vue中增加

```vue
<script setup lang="ts">
</script>

<template>
  <n-config-provider>
    <app-provider>
      <router-view />
    </app-provider>
  </n-config-provider>
</template>

```

### 测试
接下来我们分别测试一下这几个弹框是否生效，我们在pages/index.vue中进行测试。
```vue
<script lang="ts" setup>
const message = useMessage()
const dialog = useDialog()
const notification = useNotification()
const loadingBar = useLoadingBar()
const onMessage = () => {
  message.success('Hello, world!')
}

const onDialog = () => {
  dialog.success({
    title: 'Hello, world!',
    content: 'This is a success dialog.',
    onPositiveClick: onMessage,
  })
}

const onNotification = () => {
  notification.success({
    title: 'Hello, world!',
    content: 'This is a success notification.',
  })
}

const onLoading = () => {
  loadingBar.start()
  setTimeout(() => {
    loadingBar.finish()
  }, 3000)
}
</script>

<template>
  <div>
    <n-space>
      <n-button @click="onMessage">
        Message
      </n-button>

      <n-button @click="onDialog">
        Dialog
      </n-button>

      <n-button @click="onNotification">
        Notification
      </n-button>

      <n-button @click="onLoading">
        Loading
      </n-button>
    </n-space>
  </div>
</template>

<style scoped>

</style>

```
