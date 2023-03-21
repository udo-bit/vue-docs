### 目标
完成全局多语言配置
## 开发
我们在开发的过程中避免不了会有使用多语言的情况，那么接下来我们就一起配置一下多语言的功能。
### 使用
在src目录下创建一个locales的文件夹，用于存放我们的多语言。再创建一个index.ts作为我们多语言的出口。
```typescript
import { createI18n } from 'vue-i18n'

export const defaultLocale = 'zh-CN'

const i18n = createI18n({
  // 是否启用传统模式，默认true，我们这里新项目我们不需要
  legacy: false,
  // 本地化语言获取失败的时候是否输出警告
  missingWarn: false,
  // 默认多语言
  locale: defaultLocale,
  messages: {
  },

})

export default i18n

```

### 导出
在main.ts中进行导出我们的多语言：

```typescript
import i18n from '~/locales'
app.use(i18n)
```
### 配置naive多语言
接下来我们来配置一下naiveui的默认多语言，我们先在locales下面创建一个lang的文件夹，用于存放我们的多语言部分。我们以中英文为例子，如果您还需要其他的多语言请自行添加配置。
接下来我们在lang中添加多语言文件分别为:en-US.ts何zh-CN.ts文件
en-US.ts
```typescript
import { dateEnUS, enUS } from 'naive-ui'

export default {
  naiveUI: {
    locale: enUS,
    dateLocale: dateEnUS,
  },
}
```
zh-CN.ts
```typescript
import { dateZhCN, zhCN } from 'naive-ui'

export default {
  naiveUI: {
    locale: zhCN,
    dateLocale: dateZhCN,
  },
}
```
然后在locales/index.ts中导入
```typescript
import { createI18n } from 'vue-i18n'
import zhCN from '~/locales/lang/zh-CN'

export const defaultLocale = 'zh-CN'

const i18n = createI18n({
  // 是否启用传统模式，默认true，我们这里新项目我们不需要
  legacy: false,
  // 本地化语言获取失败的时候是否输出警告
  missingWarn: false,
  // 默认多语言
  locale: defaultLocale,
  messages: {
    'zh-CN': zhCN,
  },
})

export default i18n
```

然后我们创建一个组合式的api用于处理我们的多语言，在composables下创建一个auto-lang.ts的文件
```typescript
import i18n, { defaultLocale } from '~/locales'

export const useAppLocale = createGlobalState(() => useStorage('locale', defaultLocale))
export const useAutoLang = () => {
  const appLocale = useAppLocale()
  const targetLocale = computed(() => i18n.global.getLocaleMessage(appLocale.value).naiveUI)

  return {
    targetLocale,
  }
}

```
然后在App.vue中使用
```vue
<script setup lang="ts">
const appStore = useAppStore()
const { layoutTheme, overridesTheme } = storeToRefs(appStore)
useAutoDark()
+ const { targetLocale } = useAutoLang()
</script>

<template>
  <n-config-provider
+    :locale="targetLocale.locale"
+    :date-locale="targetLocale.dateLocale"
    :theme="layoutTheme"
    :theme-overrides="overridesTheme"
  >
    <n-global-style />
    <app-provider>
      <router-view />
    </app-provider>
  </n-config-provider>
</template>

```

然后我们发现控制台会报如下的警告：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10377041/1668940070155-5ca02669-b10f-4982-9a6c-ad5c160591bb.png#averageHue=%2366819d&clientId=u524bb818-f7b3-4&from=paste&height=113&id=ua68bda95&name=image.png&originHeight=226&originWidth=3280&originalType=binary&ratio=1&rotation=0&showTitle=false&size=65016&status=done&style=none&taskId=u6f4bc1e5-683d-4687-874f-9fee7f26833&title=&width=1640)
本身我们是需要使用esm的方式去构建的，所以我们在vite.config.ts中做如下的配置：
```typescript
export default defineConfig({
  define: {
    __VUE_I18N_FULL_INSTALL__: false,
    __VUE_I18N_LEGACY_API__: false,
  },
})
```
然后我们重启项目，发现我们的报错就没有了。

### 动态加载多语言
一次性加载所有的多语言是多余且没有必要的，所以接下来我们来实现一下动态加载我们的多语言的功能。
官方也提供了一个动态加载的例子[lazy loading](https://vue-i18n.intlify.dev/guide/advanced/lazy.html)
我们在locales/index.ts中增加如下代码：
```typescript

export const loadLanguageAsync = async (lang: string = defaultLocale) => {
  const current = i18n.global.locale.value
  if (current !== lang) {
    const messages = await import(`./lang/${lang}.ts`)
    i18n.global.setLocaleMessage(lang, messages.default)
  }

  return nextTick()
}
```

然后我们在auto-lang中实现设置多语言。
首先我们可以先获取我们当前的系统语言，如果当前系统语言是其他语言那么我们默认加载其他的语言。
获取当前系统的语言我们可以通过vueuse中的useNavigatorLanguage的组合式api进行获取，实现代码如下：
```typescript
import i18n, { defaultLocale, loadLanguageAsync } from '~/locales'

export const useAppLocale = createGlobalState(() => useStorage('locale', defaultLocale))
export const useAutoLang = () => {
  const appLocale = useAppLocale()
  const { isSupported, language } = useNavigatorLanguage()
  const setLanguage = async (lang: string) => {
    try {
      await loadLanguageAsync(lang)
      appLocale.value = lang
    }
    catch (e) {
      throw new Error(`Failed to load language: ${lang}`)
    }
  }
  if (isSupported.value) {
    if (language.value !== defaultLocale)
      setLanguage(language.value!).then(() => {})

    watch(language, () => {
      setLanguage(language.value!).then(() => {})
    })
  }
  else {
    if (appLocale.value !== defaultLocale)
      setLanguage(appLocale.value).then(() => {})
  }
  watch(appLocale, () => {
    if (appLocale.value !== i18n.global.locale.value)
      setLanguage(appLocale.value).then(() => {})
  })
  const targetLocale = computed(() => i18n.global.getLocaleMessage(appLocale.value).naiveUI || {})

  return {
    targetLocale,
    setLanguage,
  }
}

```

测试多语言切换，在pages/index.vue中
```vue
<script lang="ts" setup>
const appLocale = useAppLocale()
const onSwitch = (lang: string) => {
  appLocale.value = lang
}
</script>

<template>
  <div>
    <n-space>
      <n-input />
      <n-button @click="onSwitch('en-US')">
        English
      </n-button>
      <n-button @click="onSwitch('zh-CN')">
        中文
      </n-button>
    </n-space>
  </div>
</template>

<style scoped>

</style>

```
测试切换多语言，当我们切换的时候我们发现会报错：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10377041/1668941781141-eb0c3520-e4a4-4e5c-a64d-5ec72ed7de51.png#averageHue=%23351f1f&clientId=u524bb818-f7b3-4&from=paste&height=259&id=u2b1b1bc2&name=image.png&originHeight=518&originWidth=1382&originalType=binary&ratio=1&rotation=0&showTitle=false&size=157224&status=done&style=none&taskId=u15e5ee6f-5839-497a-83bf-ea6c5d4c13d&title=&width=691)
我们在auto-lang中添加如下代码：
```typescript
const targetLocale = computed(() => i18n.global.getLocaleMessage(appLocale.value).naiveUI || {})
```
然后我们就不在报错了。
多语言配置完成。
