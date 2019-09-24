---
layout:     post
title:      "VS中使用Snippets"
subtitle:   " \"VS中使用Snippets\""
date:       2019-08-05 14:44:00
author:     "Conerlius"
category: 其他
keywords: CSharp,tool
tags:
    - Tool
    - Csharp
---

> 具体的知识点可以自行在[https://docs.microsoft.com/zh-cn/visualstudio/ide/walkthrough-creating-a-code-snippet?view=vs-2019](https://docs.microsoft.com/zh-cn/visualstudio/ide/walkthrough-creating-a-code-snippet?view=vs-2019)查看，这里仅仅只是做一个简单的说明
# 开篇
> visual studio 的snippets文件模板可以通过`工具->代码片段管理器`，展示的窗体里不同语言路径不同，，自定义的一般在用户的文档下
![png](/images/snippets_for_vs-1.png)
>
> 文件必须是后缀`.snippet`
# 模板
```
<CodeSnippets>
	<!-- 1+,CodeSnippet -->
    <CodeSnippet>
		<!-- 1,Header -->
		<Header>
			<!-- 1,在管理器上的名称 -->
			<Title>... </Title>
			<!-- 作者 -->
			<Author>... </Author>
			<!-- 在管理器上代码段说明 -->
			<Description>... </Description>
			<!-- 详细信息的 URL -->
			<HelpUrl>... </HelpUrl>
			<!-- 1+,标签 -->
			<SnippetTypes>... </SnippetTypes>
			<!-- 1+,标签 -->
			<Keywords>... </Keywords>
			<!-- 使用该代码块的快捷键 -->
			<Shortcut>... </Shortcut>
		</Header>
		<!-- 1, Snippet -->
		<Snippet>
			<!-- [0,1], 声明需要引用的dll库 -->
			<References>
				<!-- 0+, 引用的dll库 -->
				<Reference>
					<!-- 1, 引用的dll库 -->
					<Assembly>... </Assembly>
					<!-- [0,1], 引用的dll库的信息link -->
					<Url>... </Url>
				</Reference>
			</References>
			<!-- [0,1], 声明需要引用的namespace -->
			<Imports>
				<!-- 0+, namespace -->
				<Import>
					<!-- 1,namespace -->
					<Namespace>System.Windows.Forms</Namespace>
				</Import>
			</Imports>
			<!-- [0,1] -->
			<Declarations>
				<!-- 0+, 声明code里使用的keyword -->
				<Literal Editable="true/false">
					<!-- 1， keyword的名称 -->
				   <ID>... </ID>
				   <!-- [0,1], 提示 -->
				   <ToolTip>... </ToolTip>
				   <!-- 1, 该关键词在输出的时候的默认文本 -->
				   <Default>... </Default>
				   <!-- [0,1], vs默认的几个元素，下文说 -->
				   <Function>... </Function>
				</Literal>
				<!-- 0+, 类似Literal -->
				<Object Editable="true/false">
					<ID>... </ID>
					<Type>... </Type>
					<ToolTip>... </ToolTip>
					<Default>... </Default>
					<Function>... </Function>
				</Object>
			</Declarations>
			<!-- 1,插入到文档文件中的代码 -->
			<CodeLanguage="Language"
				Kind="method body/method decl/type decl/page/file/any"
				Delimiter="Delimiter">
				<![CDATA[ 输出的文本,可是用Literal声明的keyword ]]>
			</Code>
		</Snippet>
	</CodeSnippet>
</CodeSnippets>
```

# 说明
1. 0+：表示零个或零个以上
2. [0,1]：表示零个或一个
3. 1：表示仅只有一个
4. 1+：表示有一个或一个以上

# Demo
```
<CodeSnippets xmlns="http://schemas.microsoft.com/VisualStudio/2005/CodeSnippet">
    <CodeSnippet Format="1.0.0">
        <Header>
            <Title>Common constructor pattern</Title>
            <Shortcut>ctor</Shortcut>
            <Description>Code Snippet for a constructor</Description>
            <Author>Microsoft Corporation</Author>
            <SnippetTypes>
                <SnippetType>Expansion</SnippetType>
            </SnippetTypes>
        </Header>
        <Snippet>
            <Declarations>
                <Literal>
                    <ID>type</ID>
                    <Default>int</Default>
                </Literal>
                <Literal>
                    <ID>name</ID>
                    <Default>field</Default>
                </Literal>
                <Literal default="true" Editable="false">
                    <ID>classname</ID>
                    <ToolTip>Class name</ToolTip>
                    <Function>ClassName()</Function>
                    <Default>ClassNamePlaceholder</Default>
                </Literal>
            </Declarations>
            <Code Language="csharp" Format="CData">
                <![CDATA[
                    public $classname$ ($type$ $name$)
                    {
                        this._$name$ = $name$;
                    }
                    private $type$ _$name$;
                ]]>
            </Code>
        </Snippet>
    </CodeSnippet>
</CodeSnippets>
```