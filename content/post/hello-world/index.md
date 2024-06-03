---
title: 你好，世界
slug: hello-world
description: 博客诞生，使用 Hugo 强力驱动。本文章包含语法测试。
image: cover.webp
date: 2024-01-01T00:00:00+08:00
categories: [Technology]
tags: [blog, hugo]
toc: true
math: true
mermaid: true
comments: false
hidden: false
draft: false
---

## 格式

- **粗体**
- _斜体_
- ~~删除线~~
- <mark>标记</mark>
- <abbr title="原文">缩写</abbr>
- 下标：H<sub>2</sub>O
- 上标：e<sup>iπ</sup> = -1
- 键盘：<kbd>CTRL</kbd> + <kbd>ALT</kbd> + <kbd>Delete</kbd>

## 链接

- [相对链接](../../about/)
- [绝对链接](/about/)
- [文章连接](../hello-world)

## 引用

> 人被杀，就会死。[^1]
>
> \- <cite>衛宮 士郎</cite>

[^1]: 这是对应的脚注

## 代码块

```html
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
  <head>
    <meta charset="utf-8" />
    <title>Example HTML5 Document</title>
  </head>
  <body>
    <p>Test</p>
  </body>
</html>
```

## 图片

![示例图片](img.webp)

![示例图册-1](img.webp) ![示例图册-2](img.webp) ![示例图册-3](img.webp)

## 表格

| Command      | Description                                        |
| ------------ | -------------------------------------------------- |
| `git status` | List all _new or modified_ files                   |
| `git diff`   | Show file differences that **haven't been** staged |

## 勾选框

- [x] issue #799
- [ ] todo: 增加功能

## 提示

{{< notice tip >}}
小提示
{{< /notice >}}

{{< notice note >}}
笔记
{{< /notice >}}

{{< notice info >}}
醒目提醒
{{< /notice >}}

{{< notice warning >}}
警告提醒
{{< /notice >}}

## 公式

行内公式：$E=mc^2$

块状公式：

$$e^{i\pi}=\cos{x}+i\sin{x}$$

## Mermaid

{{< mermaid >}}
flowchart LR
a --> b & c --> d
{{< /mermaid >}}

## 视频

{{< video "坂本真綾-菫.mp4" >}}

## 音频

{{< aplayer name="君に逢えたから" artist="eufonius" url="eufonius-君に逢えたから.mp3" cover="eufonius-君に逢えたから.webp" lrc-type=3 lrc="eufonius-君に逢えたから.lrc"/>}}
