---
date: 2023-11-17
title: Vocabulary Highter Plugin for Obsidian
tags: 
- Obsidian
- Typescript
categories: Development
---

最近写了一个Obsidian插件 [Vocabulary Highlighter](https://github.com/eatgrass/obsidian-vocab-highlighter)，按词频给文章中的单词标记不同的高亮。

## 分词

常见的英文分词方式可以参考 Python 的 `nltk` 库，自然语言处理并非我熟悉的领域，就不展开来说了。

从实际需求出发，分词需要满足以下两点需求:

1. 分出来的每个词需要贴近字典的里词条的格式 
2. 空格和标点不能丢

所以我使用了 [Treebank](https://en.wikipedia.org/wiki/Treebank) 的分词方式

## 词频和语料库

美式英语主要参考了两个语料库

- COCA (Corpus of Contemporary American English) [^1]
- ANC (American National Corpus) [^2]

COCA 是目前最权威的美式英语语料库，更新也频繁，但是需要付费使用。
ANC 语料收录与更新频次均不及 COCA, 胜在可以免费使用。

所以选择了后者，在 Open ANC 的基础上做了按词频排序的词典，词典中大概收录了24万个英文词汇。

## 文档高亮

对每个段落进行分词处理，查询词频再套上一层 `span`, 并更新 DOM, 并应用样式，最基本的高亮功能就完成了。

## 性能

1. 缓存高频词汇,可以显著减少对词典文件的查询。
2. DOM 元素批量更新,为整个段落创建 `DocumentFragment`, 将每个单词的 `span` 元素追加到 `DocumentFragment上`, 再替换原来的文本节点
3. 重新组装 DOM 元素时选择尽量不要去改变原始文档的高度，这个涉及到浏览器 `Forced Reflow` 的话题，这个会带来相当大的性能影响。
4. Obsidian 的 Markdown 后处理（Post Processor）[^3]未提供类似 CodeMirror 中视口（View Port[^4]）更新的方式，否则部分更新会带来更好的


[^1]: English-Corpora: [COCA](https://www.english-corpora.org/coca/)
[^2]: [Open American National Corpus](https://anc.org)
[^3]: Obsidian Document [Markdown post processing](https://docs.obsidian.md/Plugins/Editor/Markdown+post+processing)
[^4]: Obsidian Document [View Port](https://docs.obsidian.md/Plugins/Editor/Viewport)
