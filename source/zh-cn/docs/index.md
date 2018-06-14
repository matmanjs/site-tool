title: 概述
---

欢迎使用 matman，本文档将帮助您快速上手。如果您在使用过程中遇到问题，请在 [GitHub](https://github.com/matmanjs/matman/issues)上提问。

## 什么是 matman？

matman是本地模拟服务测试助手

能够做到：

强大的 mock 数据管理
轻易实现与native等场景的数据交互
数据校验
数据过滤和展示
可配合前端自动化测试

## 如何使用

matman 提供 `run` 方法来启动，接受一个配置参数

```
var matman = require('matman');

var options = {};

matman.run(options);
```

配置参数支持的字段和属性如下表所示。

| 字段名 | 是否必须 | 类型 | 含义描述 |
| --- | --- | --- | --- |
| `ROOT_PATH` | 是 | `String` | 项目根目录 |
| `SRC_PATH` | 否 | `String` | 源文件的目录，默认值为 `${ROOT_PATH}/src` |
| `APP_PATH` | 否 | `String` | 运行目录，默认值为 `${ROOT_PATH}/app` |
| `HANDLERS_RELATIVE_PATH` | 否 | `String` | `handler` 文件相对源文件目录的路径，默认值为 `'./handler'` |
| `DATA_PATH` | 否 | `String` | 配置数据缓存路径，默认值为 `${ROOT_PATH}/data` |
| `LOG_PATH` | 否 | `String` | 日志文件存储的路径，默认值为 `${ROOT_PATH}/logs` |
| `port` | 否 | `Number` | 服务端口号，默认为 `3000` |

更多使用请看 [demo](https://github.com/helinjiang/matman/tree/master/demo)。

## 技术细节

matman 操作的基本单位为 `mocker`，每一条 CGI 的操作，我们称之为一个 `mocker`，每一种数据返回我们称之为一个 `mock module`。

每个项目中可能有多个 `mocker` （因为会有多条CGI），每个 `mocker` 包含多个 `mock module`（因为每条CGI返回肯定会有多种状态）。

## mocker

matman 提供了一套路由系统，用于将 CGI 和 本地的 mocker 建立联系。每个 mocker 必须包含一个 `config.json` 文件，详细说明如下：

| 字段名 | 是否必须 | 类型 | 含义描述 |
| --- | --- | --- | --- |
| `route` | 是 | `String` | 路由 |
| `routeExtra` | 否 | `Object` | 对象，路由匹配的额外定义，一旦定义了这个字段，则不仅校验 `route`，请求参数还要满足 `routeExtra` 的限制 |
| `disable` | 否 | `Boolean` | 如果该值为 `true`，则会直接请求现网数据，不再使用 mock 服务了 |
| `params` | 否 | `Array` | 数组，如果路由中包含了参数，则必须定义它。例如 `/a/b/id/:id`，则需要定义 `[{"name": "id","type": "number"}]` |
| `method` | 否 | `String` | 请求类型：`get`、`post` 等， 默认为 `get` |
| `version` | 否 | `String` | 版本号 |
| `description` | 否 | `String` | 描述，默认值为 `name` 字段的值 |
| `author` | 否 | `String` | 作者 |
| `name` | 否 | `String` | 名字，默认为文件名 |
| `defaultModule` | 否 | `String` | 默认的 mock module 模块，如果不填，则会按文件名取第一个 mock module |
| `priority` | 否 | `Number` | 默认为 `0`，值越大，则会展现在列表最前面，如果值一样，则顺序不固定 |
| `tags` | 否 | `Array` | 默认为 `['全部']`，标签 |

其中必须配置的字段为 `route`。matman 是基于 [Express](http://expressjs.com/) 开发的，因此这里的 `route` 值最终将被应用到 `app.use(route,callback)` 中，而 `callback` 回调返回最终将引用到对应的 mocker 中的文件。

具体使用细节可以参考 [demo](https://github.com/helinjiang/matman/tree/master/demo)。

matman 项目启动之后，会在每个 mocker 中生成一个缓存文件 `matman.json`，一般情况下你无须关注，由于 matman 的存储是使用文件夹存储的，如果你需要保持你的行为，你可以保存 `matman.json` ，但请不要直接修改，而是使用 `config.json` 文件。

## mock module

### 必须符合 CommonJS 规范

matman 的每个 mock module 都是一个独立的 js 模块，必须符合 CommonJS 规范。

方式1：返回 `Object`：

```
module.exports = {
  "isExist": true,
  "name": "matman",
  "version": "0.0.1"
};
```

方式2：返回 `Function`：

当返回一个函数时，matman 会传递两个参数，`params` 是 CGI 的请求参数对象， `req` 是 [Express 的 req 对象](http://expressjs.com/en/4x/api.html#req)

```
export default function getData(params, req) {
    return {
        name: 'return-function-pure',
        age: params && params.age || 0
    };
}
```

如果在函数中有异步处理，则在函数中返回 `Promise` ：

```
export default function getData(params, req) {
  return new Promise((resolve, reject) => {

    // 异步处理，例如请求CGI等操作，这里使用 setTimeout 来模拟
    setTimeout(() => {
      resolve({
        name: 'return-promise',
        age: 16
      })
    }, 1000);

  });
}
```

方式3：返回 `Promise`，但是不建议使用。

### config.json 配置文件扩展能力

每个 mock module 都支持配置 `config.json` 文件，用于扩展一些能力，其字段的定义如下：

| 字段名 | 是否必须 | 类型 | 含义描述 |
| --- | --- | --- | --- |
| `version` | 否 | `String` | 版本号 |
| `description` | 否 | `String` | 描述，默认值为name字段的值 |
| `author` | 否 | `String` | 作者 |
| `name` | 否 | `String` | 名字，默认为文件名 |
| `query` | 否 | `Object` | 额外的请求参数和值，例如 `{a:1}` ，则在实际请求时，会将该参数一起发送出去 |
| `host` | 否 | `String` | 指定host，如果是伪真实数据，且需要登录态时不可缺少 |
| `priority` | 否 | `Number` | 默认为 `0`，值越大，则会展现在列表最前面，如果值一样，则顺序不固定 |


## 如何识别是 mock 数据

如果是 mock 数据，则在 response header 中会有 `matman-mocker` 和 `matman-mock-module` 字段。

![image](https://user-images.githubusercontent.com/13464168/27419496-d4aae20a-5753-11e7-86e2-cbd7048ddc3d.png)

如果禁用了整个 mocker，则在响应中增加 `matman_disable` 字段来标识，其值为 mocker 的名字。

![image](https://user-images.githubusercontent.com/13464168/27423154-453a1ffe-5763-11e7-9536-dfa0d6c4ddeb.png)


> 从请求 `req` 或者 config.json 文件中检查当前请求是否需要禁用 mock 服务，区别在于前者是单次请求（携带了 `_m_disable` 参数），而后者是所有的请求都禁用。


## 开发者文档

- [贡献代码](https://github.com/helinjiang/matman/wiki/%E8%B4%A1%E7%8C%AE%E4%BB%A3%E7%A0%81)


