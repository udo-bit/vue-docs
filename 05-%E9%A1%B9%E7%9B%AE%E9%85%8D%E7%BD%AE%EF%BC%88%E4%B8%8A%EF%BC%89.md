## 目标
1. 配置依赖库
2. 配置自动导入插件
## 依赖库配置
首先我们需要安装一下我们需要用到的UI库和工具库：[naive-ui](https://www.naiveui.com/zh-CN/os-theme)、[vueuse](https://vueuse.org/)。
然后我们还需要安装一些按需加载的插件：[unplugin-auto-import](https://github.com/antfu/unplugin-auto-import)、[unplugin-vue-components](https://github.com/antfu/unplugin-vue-components)。
路由库：[vue-router](https://router.vuejs.org/zh/introduction.html)
状态管理库：[pinia](https://pinia.vuejs.org/)
多语言：[vue-i18n](https://vue-i18n.intlify.dev/)
开启vue的[响应式语法糖](https://cn.vuejs.org/guide/extras/reactivity-transform.html#refs-vs-reactive-variables)
### 安装
```shell
pnpm add naive-ui unplugin-auto-import unplugin-vue-components -D
pnpm add @vueuse/core vue-router pinia vue-i18n

```
### 配置
首先我们需要在根目录下创建一个types的文件夹，用于存放我们的类型文件。
将src/vite-env.d.ts移动到types文件夹中，并改名为env.d.ts
接下来配置一下vite.config.ts中的插件：

```typescript
{
  plugin:[
    
    vue({
      // 响应式语法糖
      reactivityTransform: true,
    }),
    AutoImport({
      // 配置需要自动导入的库
      imports: [
        'vue',
        'vue/macros',
        'vue-router',
        'vue-i18n',
        '@vueuse/core',
        'pinia',
        {
          'naive-ui': [
            'useDialog',
            'useMessage',
            'useNotification',
            'useLoadingBar',
          ],
        },
      ],
      // 生成到的地址
      dts: 'types/auto-imports.d.ts',
      // 配置本地需要自动导入的库
      dirs: [
        // pinia状态管理目录
        'src/stores',
        // 自定义组合式api目录
        'src/composables',
      ],
    }),
    Components({
      // 导入naiveui的配置项目
      resolvers: [NaiveUiResolver()],
      // 生成类型的地址
      dts: 'types/components.d.ts',
    }),
  ]
}
```
完成配置启动项目进行测试。

我们测试一下自动导入的函数和组件的自动导入能不能正常使用。

我们发现自动导入组件可以正常使用但是没有代码提示，
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10377041/1667778607785-9a6a3184-58db-498c-bae3-388f87e28f5b.png#clientId=u6ecc8eb3-54c7-4&from=paste&height=201&id=u3f2d3be8&name=image.png&originHeight=201&originWidth=517&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34378&status=done&style=none&taskId=uebf23678-a98b-4779-b65d-e64567586c5&title=&width=517)
我们看到types/components.d.ts的目录中发现，插件使用的是@vue/runtime-core来实现的类型，那么我们在开发环境下也安装一下：

```bash
pnpm add @vue/runtime-core -D
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10377041/1667778640792-6985ac06-3d20-4384-b800-d2bbacf09f0d.png#clientId=u6ecc8eb3-54c7-4&from=paste&height=119&id=u7a8aa5dc&name=image.png&originHeight=119&originWidth=417&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12534&status=done&style=none&taskId=u65035269-b457-42eb-b2c1-b65e981ffe1&title=&width=417)
启动项目可以正常提示了。

当我们测试使用的时候发现函数没有自动导入的提示，我们看一下tsconfig.json中，在include中加上
```json
{
 "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue","types/**/*.d.ts"],
}
```
`"types/**/*.d.ts"`我们再来试一下。可以正常使用了。
