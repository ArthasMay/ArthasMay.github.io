---
title: VSCode的Vue开发环境配置
p: vscode/vscode-01
date: 2019-10-16 09:56:32
tags:
- 前端开发环境
categories: 环境配置
---


## 添加快捷代码块

### IDE配置

1. （cmd + ,）打开配置setting.json 写入下面配置
``` json
// 控制代码片段是否与其他建议一起显示及其排列的位置。
"editor.snippetSuggestions": "top",
// 控制编辑器是否应自动设置粘贴内容的格式。格式化程序必须可用并且能设置文档中某一范围的格式。
"editor.formatOnPaste": true
```

### 操作流程

1. cmd + shift + p 打开控制器
2. 选择 Configure User Snippets 创建代码块

### 模板解析

* prefix snippet的快捷缩写
* body 代码块主题

### 常用的snippts
``` html (vue)
"Scaffold": {
	"prefix": "vue",
	"body": [
		"<template>",
		"  <div class=\"$0\">\n",
		"  </div>",
		"</template>\n",
		"<script type=\"text/javascript\">",
		"export default {",
		"  data() {",
		"    return {\n",
		"    }",
		"  },",
		"  components: {\n",
		"  }",
		"}",
		"</script>\n",
		"<style lang=\"stylus\" scoped>",
		"</style>",
		"$2"
	],
	"description": "Scaffold for vue page"
}
```
