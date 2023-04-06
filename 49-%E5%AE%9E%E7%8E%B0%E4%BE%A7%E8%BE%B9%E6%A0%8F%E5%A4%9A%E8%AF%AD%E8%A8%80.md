<a name="DL33g"></a>
## 目标
实现侧边栏动态多语言渲染。
<a name="ccfyU"></a>
## 开发
首先我们需要将我们的请求菜单的api切换到我们的多语言的mock数据中，方便我们去实现。<br />在api/user.ts中
```typescript
// export const userRoutesUrl = '/user/menus'
export const userRoutesUrl = '/user/menu-lang'
```
切换到多语言的数据后，我们会发现我们的菜单发生变化了，接下来我们根据我们菜单的数据，去实现一下我们的菜单。<br />首先我们在pages下创建一个dashboard目录。然后分别创建一个analysis.vue和workspace.vue的文件。
```vue
<script lang="ts" setup>

</script>

<template>
  <div>
    Analysis
  </div>
</template>

<style scoped>

</style>

```
```vue
<script lang="ts" setup>

</script>

<template>
  <div>
    Workspace
  </div>
</template>

<style scoped>

</style>
```
然后我们在routes/modules中创建一个dashboard.ts的文件：
```typescript
const DashboardAnalysis = () => import('~/pages/dashboard/analysis.vue')
const DashboardWorkspace = () => import('~/pages/dashboard/workspace.vue')
export default {
  DashboardAnalysis,
  DashboardWorkspace,
}

```
然后我们发现配置完成后页面不能正常访问了，是因为我们还需要调整一下我们的dynamic-routes.ts的默认重定向的路由地址：
```typescript
export const rootRouter: RouteRecordRaw = {
  path: '/',
  name: 'default-router',
  redirect: '/dashboard',
  component: Layout,
  children: [],
}
```
然后我们再刷新，会发现我们现在就是正确的页面地址了。<br />现在给我们返回的title是多语言的title，所以我们还是需要先配置一下基础的多语言。<br />在locales/pages下分别配置zh-CN和en-US
```typescript
	'pages.dashboard.title': 'Dashboard',
  'pages.dashboard.analysis.title': 'Analysis',
  'pages.dashboard.workspace.title': 'Workspace',
  'pages.jump.baidu': 'Jump to Baidu',
```
```typescript
  'pages.dashboard.title': '仪表盘',
  'pages.dashboard.analysis.title': '分析页',
  'pages.dashboard.workspace.title': '访问量',
  'pages.jump.baidu': '跳转百度',
```
最后我们来实现一下多语言的部分在side-title.vue中：
```vue
<template v-if="hasChildren">
    {{ $t(title) }}
  </template>
  <template v-else-if="isFullPath">
    <a :href="path" :target="target">{{ $t(title) }}</a>
  </template>
  <template v-else>
    <router-link :to="path">
      {{ $t(title) }}
    </router-link>
  </template>
```
查看效果。<br />最后我们发现，标题之前我们配置了多语言，所以导致了我们的标题的部分现在显示的也是多语言的key，所以我们这一部分也需要调整一下。<br />在routes/router-guard.ts中：
```typescript
const title = to.meta?.title
if (title) {
+  const localeTitle = i18n.global.t(title)
  document.title = `${localeTitle} - ${appStore.layout.title}`
}
```
但是我们发现我们切换多语言的时候不能动态的更新标题，那么接下来我们来实现一下切换动态更新标题。<br />在composable/auto-lang.ts中：
```typescript
const { locale, getLocaleMessage, t } = useI18n()
const route = useRoute()
const appStore = useAppStore()
const setLanguage = async (lang: string) => {
  try {
    await loadLanguageAsync(lang)
    appLocale.value = lang
    locale.value = lang
    const title = route.meta.title
    if (title) {
      const localeTitle = t(title)
      document.title = `${localeTitle} - ${appStore.layout.title}`
    }
  }
  catch (e) {

  }
}
```
然后再进行测试。