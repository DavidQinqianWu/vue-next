---
description: 个人Vue源码的学习心得总结
---

# Vue 源码学习笔记

## 1. 环境调试运行

### 1.1 npm 运行文件分析\(`/scrips/dev.js`\)

* 从 /scripts 可以获得在 `npm run dev`是的时候回调用 dev.js 的代码逻辑,
* `const args = require('minimist')(process.argv.slice(2))` 这个中的\(`minimist`\)库可以允许我们在设置 运行指令的时候 添加 -v -f 等等自定的属性参数,

```javascript
$ node example/parse.js -x 3 -y 4 -n5 -abc --beep=boop foo bar baz
// minimist 库可以把我们的指令给转成这样的模式
{
  _: [ 'foo', 'bar', 'baz' ],
  x: 3,
  y: 4,
  n: 5,
  a: true,
  b: true,
  c: true,
  beep: 'boop' 
}
```

```javascript
const args = require('minimist')(process.argv.slice(2)) // 分解指令
const target = args._.length ? fuzzyMatchTarget(args._)[0] : 'vue'
const formats = args.formats || args.f // 把-f 或者 -formats 指令中的参数提取出来
const sourceMap = args.sourcemap || args.s // 把 -s 或者 -sourcemap 中的参数提取出了, 在 rollup 打包的时候, vue3 根据你这个参数决定是否生成 sourcemap
execa(
  // execa 是用来在 node 环境的时候是赋值给全局变量
  'rollup',
  [
    '-wc',
    '--environment',
    [
      `COMMIT:${commit}`,
      `TARGET:${target}`,
      `FORMATS:${formats || 'global'}`,
      sourceMap ? `SOURCE_MAP:true` : ``
    ]
      .filter(Boolean)
      .join(',')
  ],
  {
    stdio: 'inherit'
  }
)
```

### 1.2 git commit 文件分析\(`/scrips/verifyCommit.js`\)

* 在我们自己 fork 了代码之后, 如果提交代码没有按照 vue3 的 commit 要求规范是无法被推送的, 那么这个控制你的 commit 的格式就是在 `/scripts/verifyCommit.js` 文件中

```javascript
// Invoked on the commit-msg git hook by yorkie.

const chalk = require('chalk')
const msgPath = process.env.GIT_PARAMS
const msg = require('fs')
  .readFileSync(msgPath, 'utf-8')
  .trim()

const commitRE = /^(revert: )?(feat|fix|docs|dx|style|refactor|perf|test|workflow|build|ci|chore|types|wip|release)(\(.+\))?: .{1,50}/
// 从这里可以看出 对我们的品论提交 有严格的要求, 除了不能有 需要包含以上的评论, 例如:
// test: this is test
// 当然 不能有 revert, 在代码里面已经给出了

if (!commitRE.test(msg)) {
  console.log()
  console.error(
    `  ${chalk.bgRed.white(' ERROR ')} ${chalk.red(
      `invalid commit message format.`
    )}\n\n` +
      chalk.red(
        `  Proper commit message format is required for automated changelog generation. Examples:\n\n`
      ) +
      `    ${chalk.green(`feat(compiler): add 'comments' option`)}\n` +
      `    ${chalk.green(
        `fix(v-model): handle events on blur (close #28)`
      )}\n\n` +
      chalk.red(`  See .github/commit-convention.md for more details.\n`)
  )
  process.exit(1)
}
```

## 2. 核心 API

### `createApp()` 调用流程

> `createApp()`返回一个实例, 在其中首先调用了 `ensureRenderer()` 这个方法

#### 1. `ensureRenderer()`

```javascript
function ensureRenderer() {
  return (
    renderer || ((renderer = createRenderer < Node), Element > rendererOptions) // 此方式能够确保 生成唯一的 renderer 对象在全局里面
  )
}
```

> 用来确保当前存在 `renderer`对象, 如果存在就返回, 如果不存在就调用`createRenderer()`方法, 接收 `renderOptions`作为参数

#### 2. `createRenderer()`

> 这里面调用了 `baseCreateRenderer()`, 传入 options,是一个对象, interface 是`RendererOptions`,`RendererOptions`里面有很多方法,是用来给做虚拟 dom 用的方法.

#### 3. `baseCreateRenderer()`

> `vnode`, `diff`, `patch`等方法军事在这个函数里面去实现 返回是 `render()`, `hydrate()` 和 `createApp()`, `createApp()`调用了 `createAppAPI(render, hydrate)`

#### 4. `createAppAPI()`

> 返回调用 _内部生成的_ `createAPP()`

```javascript
 function isObject(value){
   return value!==null && typeof val ==='object
 }
 function isFunction(value){
   return typeof value ==='function'
 }
 let  obj = {
      then:()=>{
      },
      catch:()=>{
      },

}
console.log(isObject(obj)&& isFunction(obj.then) && isFunction(obj.catch))
```

