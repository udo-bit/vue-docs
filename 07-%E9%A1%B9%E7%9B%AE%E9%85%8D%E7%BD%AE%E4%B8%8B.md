## 目标
1. 配置路径前缀
2. 创建环境变量文件

## 配置路径前缀
我们经常会在一些项目框架中会看到类似下面的引用路径：
```typescript
import App from "~/App.vue"
```
那么他是怎么通过 “~”来配置路径的呢。
vite为我们提供了alias的系统别名的属性。我们可以通过他来配置。
在vite.config.ts中
> 我们在使用fileURLToPath的时候没有提示，我们需要安装一下node的类型库  pnpm add @types/node -D

```typescript
const baseSrc = fileURLToPath(new URL('src', import.meta.url));
export default {
   // ...省略其他
  resolve:{
    alias:{
      "~":baseSrc,
      "~@":baseSrc
    }
  }
}
```
配置完成后，我们在项目中尝试引用，发现并没有路径提示，那么我们还需要在tsconfig.json中做配置如下：
```json
{
  "compilerOptions":{
    "baseUrl":".",
    "paths":{
      "~@/*": ["src/*"],
      "~/*": ["src/*"],
    }
  }
}
```
然后我们再在项目中测试发现已经生效了（如果未生效建议重启编辑器）

### 环境变量
默认情况下，vite会使用dotenv来读取一下的文件，作为我们的环境变量。
```bash
.env                # 所有情况下都会加载
.env.local          # 所有情况下都会加载，但会被 git 忽略
.env.[mode]         # 只在指定模式下加载
.env.[mode].local   # 只在指定模式下加载，但会被 git 忽略
```
[环境变量文档参考地址](https://cn.vitejs.dev/guide/env-and-mode.html#env-files)，这里我们不在赘述直接上手使用
我们在根目录下，创建 .env .env.production .env.development文件，分别用于默认配置，生产环境配置 开发环境配置。
默认情况下，为了防止意外地将一些环境变量泄露到客户端，只有以VITE_为前缀的变量才会暴露给vite处理的代码。所以我们在应用的时候保证我们的前缀是以VITE_开头即可。
例如：
```bash
VITE_APP_BASE='/'
```
我们在项目中测试一下读取这个变量
在main.ts中：
```bash
console.log(import.meta.env.VITE_APP_BASE)
```
其中import.meta.env仅仅是vite中的特性，不能在其他地方使用

### typescript提示配置
默认情况下，环境变量是没有提示的，所以我们可以通过配置类型使得我们在使用环境变量的时候有一定的提示信息，方便我们后续的开发。
在env.d.ts中加入如下配置
```typescript
interface ImportMetaEnv {
  readonly VITE_APP_BASE: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```
我们再尝试输入，就会自带提示了，我们需要哪些变量我们就配置哪些即可。
