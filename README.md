# auto-inject-async-catch 🚀

基于 babel 实现的自动注入 async 函数的 try catch 语句。基础配置可以看 [async-catch](https://github.com/yeyan1996/async-catch-loader) 讲解或者源码定义。这里，我优化了向上查找 parent 的过程，优化后的 `traverse` 如下所示：

```javascript
traverse(ast, {
    AwaitExpression(path) {
      // 已经包含 try 语句则直接退出
      if (
        path.findParent(path => t.isTryStatement(path.node))
      ) {
        return;
      }
      // 查找最外层的 async 语句
      const blockParent = path.findParent(path => t.isBlockStatement(path.node))
      const tryCatchAst = t.tryStatement(
        blockParent.node,
        t.catchClause(
          t.identifier(options.identifier),
          t.blockStatement(catchNode)
        ),
        finallyNode && t.blockStatement(finallyNode)
      )
      blockParent.replaceWithMultiple([tryCatchAst])
    }
  });
```

> 主要是它目前不怎么维护了，所以出于自己业务需求，所以自己 fork 了一份。

## 在 Vue-CLI 中使用

**1. 使用 JavaScript 开发**

使用 JavaScirpt 开发的同学只需要通过 `chainwebpack` 选项在 `js` rule 中添加一个 `loader` 就行。在 vue.config.js 的 `chainWepack` 中加入如下配置：
```javascript
chainWepack: (config) => {
  // TODO: cache-loader 的 options 后期需要完善一下
  const jsRule = config.module.rule("js");
  jsRule
    .use("auto-inject-try-catch-loader")
    .loader("auto-inject-try-catch-loader")
    .end()
}
```

**2. 使用 TypeScript **

使用 TypeScript 开发的同学需要重写整个 `ts` rule 的 loader 配置。在 vue.config.js 的 `chainWepack` 中加入如下配置：
```javascript
chainWebpack: (config) => {
  // TODO: cache-loader 的 options 后期需要完善一下
  const tsRule = config.module.rule("ts");
  tsRule.uses.clear();
  tsRule
    .use("cache-loader")
      .loader("cache-loader")
      .end()
    .use("babel-loader")
      .loader("babel-loader")
      .end()
    .use("auto-inject-async-catch-loader")
      .loader("auto-inject-async-catch-loader")
      .tap(() => {
        return {
          catchCode: 'console.error(e)'
        }
      })
      .end()
    .use("ts-loader")
      .loader("ts-loader")
      .tap(() => {
        return {
                  transpileOnly: true,
                  appendTsSuffixTo: [
                    '\\.vue$'
                  ],
                  happyPackMode: false
                }
      })
      .end()
}
```
