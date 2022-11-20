---
title: tsconfig.json文件的配置
date: 2022-10-23 21:56
tags: react
---

一个目录下存在一个tsconfig.json文件，那么它意味着这个目录是TypeScript项目的根目录。 tsconfig.json文件中指定了用来编译这个项目的根文件和编译选项。tsconfig.json文件可以是个空文件，那么以默认配置选项编译。在命令行上指定的编译选项会覆盖在tsconfig.json文件里的相应选项

```js

{
  "compilerOptions": {
     // 指定ECMAScript目标版本
    "target": "ESNext",
    "useDefineForClassFields": true,
    // TS需要引用的库，即声明文件，es5 默认引用dom、es5、scripthost,如需要使用es的高级版本特性，通常都需要配置，如es8的数组新特性需要引入"ES2019.Array",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    // 允许编译器编译JS，JSX文件
    "allowJs": false,
    // 忽略所有的声明文件（ *.d.ts）的类型检查
    "skipLibCheck": true,
    // 允许module.exports=xxx 导出，由import from 导入.
    "esModuleInterop": false,
    // 允许从没有设置默认导出的模块中默认导入
    "allowSyntheticDefaultImports": true,
    // 启用所有严格类型检查选项
    "strict": true,
    // 禁止对同一个文件的不一致的引用
    "forceConsistentCasingInFileNames": true,
     // 设置程序的模块系统
    "module": "ESNext",
    // 模块解析策略。默认使用node的模块解析策略
    "moduleResolution": "Node",
    // 允许导入扩展名为“.json”的模块
    "resolveJsonModule": true,
    // 将每个文件作为单独的模块（与“ts.transpileModule”类似）。
    "isolatedModules": true,
    // 不输出文件,即编译后不会生成任何js文件
    "noEmit": true,
    //在 `.tsx`文件里支持JSX： `"React"`或 `"Preserve"`
    "jsx": "react-jsx",
    // 解析非相对模块的基地址，默认是当前目录
    "baseUrl": "./src",
    // 模块名到基于 baseUrl的路径映射的列表
    "paths": {
      "@/*": ["./*"]
    }
  },
  //包含的文件
  "include": ["src", "vite.config.ts"],
//   每个引用的path属性都可以指向到包含tsconfig.json文件的目录，或者直接指向到配置文件本身（名字是任意的）
  "references": [{ "path": "./tsconfig.node.json" }]
}

```