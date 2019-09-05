---
layout:     post
title:      "Vscode中的snippets"
subtitle:   " \"Vscode中的snippets\""
date:       2019-08-05 14:38:00
author:     "Conerlius"
header-img: "img/post-bg-2015.jpg"
tags:
    - Tool
---

# snippets是什么？
> 简而言之，这个就是个代码块填充工具
# snippets能做什么？
> 1. 填充lua的注释
> 2. 生成固定的代码模板
# snippets怎么配置？
> 在vscode里通过`File->Perference->User Snippets`打开
![png](/images/snippets_for_vscode.png)
> 打开后是一系列的语言的snippets配置，没有的话会自动创建
![png](/images/snippets_ui.png)

## lua作为实例
> 刚打开的新配置是像下图的
```
{
	// Place your snippets for lua here. Each snippet is defined under a snippet name and has a prefix, body and 
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	// same ids are connected.
	// Example:
	// "Print to console": {
	// 	"prefix": "log",
	// 	"body": [
	// 		"console.log('$1');",
	// 		"$2"
	// 	],
	// 	"description": "Log output to console"
	// }
}
```
> 这里就以一个lua的注释模板作为说明
```
"Print to console": {
		 	"prefix": "autoDes",
		 	"body": [
				 "--!@brief ${1:Describe}",
				 "--!@param ${2:paramName} ${3:paramDes}",
				 "--!@return ${4:returnDes}"
		 	],
		 	"description": "自动生成lua注释模板"
		 }
```