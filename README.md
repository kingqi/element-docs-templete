# 组件库框架模板

参考element-ui项目结构和实现方案，实现的组件库项目框架模板

### 项目结构

```
├── build  // 编译脚本
├── packages // 组件代码
│	    └── form // 示例组件
│           ├── src
│           │     ├── config.js
│           │     └── form.vue
│           └── index.js
├── examples // 说明文档
│     ├── assets  // 资源文件
│     ├── components // vue组件
│     ├── docs // 每个组件写一个MD文档在这里
│     ├── pages // vue页面文件
│     ├── app.vue // 根文件
│     ├── entry.js // 入口
│     ├── nav.config.json // 路由数据
│     └── util // 工具函数（只有说明文档使用
├── src
│     ├── styles  // 组件公共样式 
│     ├── mixinx  // vue混入 
│     ├── utils // 通用工具库 
│     └── index.js // 组件库入口（自动生成） 
└── components.json // 所有组件的列表，用于自动生成入口
```
