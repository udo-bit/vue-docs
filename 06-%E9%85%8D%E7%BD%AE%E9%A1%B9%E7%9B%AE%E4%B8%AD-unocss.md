## 目标
- 安装配置unocss

### 什么是unocss
unocss是一个原子CSS引擎，而不是一个框架。一切的设计都考虑到了灵活性和性能。unocss中没有核心实用程序，所有功能都是通过预设提供的。
### 关于预设
预设作为Unocss的核心。利用预设我们可以创建我们自己的自定义框架。
默认情况下，Unocss应用默认预设，它提供了流行使用程序优先框架Tailwind CSS、Windi CSS、Bootstrap等的通用的超集。
也就是说如果你之前对以上的框架比较熟悉，那么你在unocss中可以照常使用他们的语法即可。

### 特性

- 完全可定制
- 没有解析，没有AST，没有扫描，即时编辑。
- 包体积小
- 动态别名
- 属性模式
- 纯CSS图标
- CSS Directives
- 编译模式
- Inspector
- CSS in JS runtime Build
- CSS 的代码分割

### 文档地址
unocss提供了一个[在线搜索](https://uno.antfu.me/)可转换的地址，但是没有学习语法的文档，如果想要学习的话，可以使用[Tailwind](https://www.tailwindcss.cn/docs/installation)的文档，因为大部分他们的用法都是一致的。不过大家可以跟着我一起，通过这个项目我会带着大家一起去使用我们的unocss，帮助大家快速上手。

## 安装
这里我们需要安装unocss和重置默认样式的工具@unocss/reset 

```bash
pnpm add unocss @unocss/reset -D
```
### 使用
在vite.config.ts中导入插件
```bash
import Unocss from 'unocss/vite'
{
	plugins:[Unocss()]
}
```

我们将unocss的配置项单独抽离出来在根目录创建一个unocss.config.ts文件，导入一些常用默认的预设

unocss.config.ts
```typescript
import {
  defineConfig,
  presetAttributify,
  presetIcons,
  presetTypography,
  presetUno,
  presetWebFonts,
  transformerDirectives,
  transformerVariantGroup,
} from 'unocss'

export default defineConfig({
  shortcuts: [],
  presets: [
    presetUno(), // 默认wind预设
    presetAttributify(), // class拆分属性预设
    presetTypography(), // 排版预设
    presetIcons({ // 图标库预设
      scale: 1.2,
      warn: true,
    }),
    presetWebFonts({ // 网络字体预设
      fonts: {
        sans: 'DM Sans',
        serif: 'DM Serif Display',
        mono: 'DM Mono',
      },
    }),
  ],
  transformers: [
    transformerVariantGroup(), // windi CSS的变体组功能
    transformerDirectives(), //  @apply @screen theme()转换器
  ],
})
```

在main.ts中导入

```typescript
import "@unocss/reset/tailwind.css"
import "uno.css";
```

需要注意的是。默认情况下，tailwind的重置规则会覆盖naive-ui的规则，所以我们需要让naive-ui的优先级最高，那么我们需要再main.ts中添加如下代码
```typescript
/**
 * 解决tailwind的样式冲突
 */
const meta = document.createElement('meta')
meta.name = 'naive-ui-style'
document.head.appendChild(meta)
```
[参考地址-潜在的样式冲突](https://www.naiveui.com/zh-CN/os-theme/docs/style-conflict)

我们在项目中进行测试。
