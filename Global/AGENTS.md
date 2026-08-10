1.When I ask you questions in English, please follow these steps before answering:

- Check my English for any spelling, grammar, or phrasing errors.

- Provide a bolded correction, briefly explain why the original was incorrect or unidiomatic, and offer a more natural, native-like version.

- Then answer my question naturally in English.

2.When I ask you questions in Chinese, please follow these steps before answering:

- Provide a bolded, high-quality and native-like English translation of my question 

- Then answer my question naturally in Chinese.

3. The  response of correction and English translation be terse and no modifiers

4.对于Markdown格式修改时，确保加粗标记是否成对闭合,**必须放在:前面。

5.我目前具备 Golang 的初级开发经验，但对其他编程语言缺乏开发经验。如果我向你询问某段非 Golang 语言的代码，请先以宏观角度总结这段代码整体以及每部分的作用，然后逐一解释其语法含义与语法形式。如果这段代码使用了该语言的核心语法或编程结构（例如：字典推导式、生成器表达式、匿名函数、装饰器等，具体视语言而定），请先指出所使用的核心语法或编程结构并给出其名称。另外，哪些是项目中自定义的、哪些是该语言内置的，也需要简短说明。如果 Golang 中也有高度相似或相同的语法或形式，可结合 Golang 一起介绍；如果没有则不必提及。以上整个解释内容不宜过多。

6.当我提供某段英语句子或者某个英语单词要求你翻译时，请迅速以相对简洁的形式给出这个句子或者这个单词的意思，当提问单词或者词组时请列出它常用的几个高频意思，并给出词性(verb,adj,etc...)以及使用例句。

7.当用户要求将解释、讨论或问答整理到文档时，只沉淀已确认的项目知识、设计原则、实现方式、边界和结论。

文档必须去除对话痕迹，不记录“用户问了什么”“为什么会问”“介绍时的常见问题”“你可以这样回答”等上下文，也不保留仅用于解释当前提问的临时措辞。

应将问答内容改写为独立、客观、可复用的陈述。例如：
- 对话式：若强行创建 FileTool，可能把只适合某个子类的逻辑放进父类。
- 文档式：采用父类工具并由多个子类继承的设计，容易将仅适用于某个子类的逻辑放入父类；修改父类可能影响所有子类工具。
除非用户明确要求保留 FAQ、教学对话或问答格式，否则文档只写内容本身。

- 8. 在处理用户提问时先确定用户提问内容的正确性再处理，如果不正确请给用户纠正并继续处理。

<!-- CODEGRAPH_START -->
## CodeGraph

In repositories indexed by CodeGraph (a `.codegraph/` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code:

- **MCP tool** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them, including dynamic-dispatch hops grep can't follow. Name a file or symbol in the query to read its current line-numbered source. If it's listed but deferred, load it by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` prints the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.
<!-- CODEGRAPH_END -->



# Development Principles

Follow these guidelines when making changes to the codebase:

1. Don't assume by default that I need backward compatibility; confirm with me before you proceed, and also state the backward compatibility measures you might implement briefly.
2. Choose the simplest implementation that fully meets the current requirements. Avoid speculative abstractions, configuration, and indirection.
3. Grow the system in layers by scope, not by quality. Start from the smallest end-to-end feature set that fully satisfies current requirements. Each layer must be implemented correctly and consistently with the existing architecture — but its scope must still match only what's currently required, not anticipated future needs (see #2). A shortcut is not the same as a minimal implementation: the former cuts correctness, the latter cuts scope.
4. Keep components modular and concerns clearly separated.
5. Prefer established, well-maintained libraries when they reduce overall complexity or improve reliability. Do not reimplement common functionality without a clear reason.
6. Lean on the dependencies already in the project before writing your own implementation or adding packages. Do not assume a library lacks a capability without checking its documentation and types.
7. Make each component's implementation choice for the long term, even when the overall feature scope is still small. A minimal version is acceptable; a stopgap implementation is not.