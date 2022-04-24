# 反馈平台

This project is initialized with [Ant Design Pro](https://pro.ant.design). Follow is the quick guide for how to use.

## Environment Prepare

Install `node_modules`:

> 使用 yarn 作为包管理器

```bash
yarn
```

## Provided Scripts

Ant Design Pro provides some useful script to help you quick start and build with web project, code style check and test.

Scripts provided in `package.json`. It's safe to modify or add additional script:

### 本地开发

```bash
npm start
```

### 构建

```bash
npm run build
```

### Check code style

```bash
npm run lint
```

You can also use script to auto fix some lint error:

```bash
npm run lint:fix
```

## 项目配置

Pro 脚手架默认使用 Umi 作为底层框架，Umi 在 .umirc.ts 或 config/config.ts 中配置项目和插件，支持 es6

如果项目的配置不复杂，推荐在 .umirc.ts 中写配置

如果项目的配置比较复杂，**就像本项目中**，将配置写到了 `config/config.ts` 中，并把配置的一部分拆分出去，比如路由配置可以拆分成单独的 routes.ts

```js
// config/routes.ts

export default [{ exact: true, path: '/', component: 'index' }];
```

然后在 config.ts 里将其引入

```js
// config/config.ts

import { defineConfig } from 'umi';
import routes from './routes';

export default defineConfig({
  routes: routes,
});
```

## 环境变量

在开发中经常会有一些需求，根据应用运行的不同环境进行不同的逻辑处理

比如，dev 环境使用 dev 的对应的 Url，而线上则使用 prod 对应的 Url。 或者，在某些特定的环境需要打开只有在该环境下才会生效的功能

### 多运行环境管理

Pro 项目中可以通过指定 `UMI_ENV` 这一环境变量来区分不同环境的配置文件，`UMI_ENV` 需要在 `package.json` 内配置，例如：

```json
{
  "scripts": {
    "analyze": "cross-env ANALYZE=1 umi build",
    "build": "umi build",
    "build:dev": "cross-env REACT_APP_ENV=dev UMI_ENV=dev umi build",
    "build:test": "cross-env REACT_APP_ENV=test UMI_ENV=test umi build",
    "build:pre": "cross-env REACT_APP_ENV=pre UMI_ENV=pre umi build",
    "build:prod": "cross-env REACT_APP_ENV=prod UMI_ENV=prod umi build",
    "deploy": "npm run site && npm run gh-pages",
    "dev": "npm run start:dev",
    "start": "cross-env REACT_APP_ENV=dev UMI_ENV=dev umi dev",
    "start:dev": "cross-env REACT_APP_ENV=dev UMI_ENV=dev MOCK=none umi dev",
    "start:no-mock": "cross-env REACT_APP_ENV=dev UMI_ENV=dev MOCK=none umi dev",
    "start:no-ui": "cross-env REACT_APP_ENV=dev UMI_ENV=dev UMI_UI=none umi dev",
    "start:pre": "cross-env REACT_APP_ENV=pre UMI_ENV=pre MOCK=none umi dev",
    "start:test": "cross-env REACT_APP_ENV=test UMI_ENV=test MOCK=none umi dev"
  }
}
```

当 `UMI_ENV` 为 test 时，必须在 config 目录下配置 config.test.js 文件来管理 test 环境下的环境变量，Umi 会在 deep merge 后生成最终配置，一个包含完整的开发、测试、预发、生产环境的 config 目录形如下面的示例：

```js
├── config
│   ├── config.dev.ts
│   ├── config.test.ts
│   ├── config.pre.ts
│   ├── config.prod.ts
│   ├── config.ts
│   ├── proxy.ts
│   ├── routes.ts
│   ├── defaultSettings.ts
```

### 获取当前运行环境名称

在 Pro 的脚手架中有这样的一个环境变量 REACT_APP_ENV，该变量代表当前应用所处环境的具体名称。如 dev、test、pre、prod 等

如若需要在 config 外的非 node 环境文件中使用该环境变量，则需要在 config 导出默认 defineConfig() 时配置 define{}，代码示例如下：

```js
// config/config.ts
const { REACT_APP_ENV } = process.env;

export default defineConfig({
  define: {
    REACT_APP_ENV: REACT_APP_ENV || false,
  },
});
```

使用该环境变量时：

```js
// src/components/RightContent/index.tsx
const GlobalHeaderRight: React.FC<{}> = () => {
/** 省略其他代码 */
return (
  <Space className={className}>
    <!-- 在顶部右侧显示对应环境名称 -->
    {REACT_APP_ENV && (
      <span>
        <Tag color={ENVTagColor[REACT_APP_ENV]}>{REACT_APP_ENV}</Tag>
      </span>
    )}
  </Space>
);
};
```

对于其他环境需要使用的变量，这里以 test 环境为例：

```js
// config/config.test.ts test环境对应的配置文件
import { defineConfig } from 'umi';

/**
 * 导出的多环境变量命名约定：一律大写且采用下划线分割单词
 * 注意：在添加变量后，需要在src/typing.d.ts内添加该变量的声明，否则在使用变量时IDE会报错。
 */
export default defineConfig({
  define: {
    API_URL: 'https://api-test.xxx.com', // API地址
    API_SECRET_KEY: 'XXXXXXXXXXXXXXXX', // API调用密钥
  },
});
```

使用该环境变量时：

```js
// src/services/user.ts
import { request } from 'umi';

export async function query() {
  // 使用API密钥调用用户接口
  return request<API.CurrentUser[]>('${API_URL}/api/users', {
    API_SECRET_KEY,
  });
}
```

### 处理报错

由于环境变量是直接使用，不会通过 window 对象的方式来使用，在 eslint 和 TypeScript 中都会报错

eslint 中可以通过增加 globals 的配置来处理报错，代码示例：

```js
{
"globals": {
  "page": true
}
}
```

而在 ts 中，需要在 `typings.d.ts` 中进行定义：

```js
// src/typings.d.ts
declare const REACT_APP_ENV: 'test' | 'dev' | 'uat' | 'prod' | undefined;
// 以下变量声明对应config.[env].ts文件内define的变量
declare const API_URL: string;
declare const API_SECRET_KEY: string;
```

## More

You can view full document on our [official website](https://pro.ant.design). And welcome any feedback in our [github](https://github.com/ant-design/ant-design-pro).