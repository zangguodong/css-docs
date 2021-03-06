# CSS Display Module Level3

## 摘要

本模块描述了，文档元素树是如何产生 CSS 格式化盒子树的，并定义了控制这个行为的 `display` 属性
CSS 是用于描述结构化文档在屏幕上，纸张等媒体上排版的语言

## 1 介绍

_本章节是规范的_
CSS 会将一份源文档，以树结构，来组织的元素和文本节点，并将该树渲染到 `canvas`（比如你的屏幕，一张纸，或者一个声频流）上。（一些源文档可能会由更复杂的树组成，比如 DOM，其可以有注释节点和其他类型的东西。对 CSS 来说，这些额外类型的节点会被忽视，就好像他们不存在一样。）
为了做到这点，会产生一个中间结构，即盒子树（box tree）。盒子树代表被渲染文档的排版结构。在 canvas 上，盒子树中的每个盒子在时间和空间上代表了的其对应的元素（或伪元素），而盒子树中的每个文本片段（text run）代表了对应的文本节点的内容。

为了创建盒子树，CSS 首先使用层叠和继承，给元素树中的每个元素和文本节点的每个 CSS 属性计算并分配一个计算值。

之后，对于每个元素，CSS 会根据元素的 display 属性产生 0 个或者一些盒子。典型的，一个元素会产生一个单独的盒子，即主盒子（principle box），该盒子代表了元素自身，并在盒子树中包含元素的内容。然而，一些 display 的值（如 'display: list-item'）会产生不止一个盒子（如：一个主块盒子，和一个子标记盒子）。一些值（如 none 或 contents）会导致元素 和/或 其后代根本不产生盒子。盒子通常通过 display 类型来指定 -- 比如，一个具有 `display: block`的元素产生的盒子叫做块盒子（block box）或者干脆叫块（block）

除非特别指定，由元素产生的盒子会和生成元素具备相同的样式。通常，继承属性会被分配给主盒子，之后通过盒子树继承给同一个元素的产生的其他盒子。非继承属性会默认应用给主盒子，但是当元素产生多个盒子时，有时会指定分配给不同的盒子。：举个例子， 应用到 table 元素上的 border 属性，会应用到其 table grid 盒子，而不是 table wrapper 盒子。如果在 CSS 属性值计算过程中改变了这些盒子的样式，并且请求获取元素的样式，（如通过 getComputedStyle），那么对于每个属性，元素将会返回这个属性真正应用在的盒子上的值。

类似的，每个相邻的文本节点会产生一个文本段（text run）来包含他们的文本内容，这个 text run 会应用文本节点的样式。如果一个片段没有任何文本，则不会产生一个 text run
构建盒子树时，一个元素产生的盒子是任意祖先元素生成的主盒子的后代。通常情况下，一个元素的主盒子的直接父盒子是它最近的祖先元素产生的主盒子。然而，有一些例外，比如对于 run-in 盒子，或者一些特殊的显示类型 （如 table）会产生多个容器盒子以及中间匿名盒子。

一个匿名盒子是指一个没有和任何元素关联的盒子。当元素树的结构不合规范时，会产生匿名盒子来修补盒子树，来得到规范的嵌套结构。举个例子，一个 table cell 盒子需要特定类型的父盒子，即 table row 盒子。因此假如这个 cell 父盒子不是 row 盒子，则会在这个 cell 之外产生一个匿名的 row 盒子，与元素生成的盒子不同，匿名盒子只能通过盒子树上的父结构继承样式，而不能直接从元素树上获得样式。
对于排版目的，盒子和文本段可以分成多个片段。这会在一些情况下发生，比如，当一个内联盒子 和/或 文本段在多行时，由于折行会被分段。或者当一个块盒子会在跨页或跨列的时候被分段，这个分段的处理过程叫做分段化（fragmentation）。这也可能是由于对文本的双向重排序引起的，或者由更高级的 display 类型盒子分割（如 block-in-inline 分割，详见 `CSS2\$9.2` 或 column-spanner-in-block 分割，详见 CSS 多列布局）。一个盒子因此可以由一个或者多个盒子片段组成，一个 text run 也可能由一个或者多个 text 片段组成更多关于分段的信息，详见 `CSS3-BREAK`
_注意：进一步关于 aural 盒子树的信息以及其与 display 的交互可以参考 `CSS Speech Module`_

### 1.1 模块交互

这个模块替换并扩展了 CSS2\$9.2.4 的 display 属性定义
本模块的所有属性都不是用于 ::first-line 或 ::first-letter 伪元素

### 1.2 值定义

本规范遵守 CSS2 的 CSS 属性定义规范，使用 `CSS-VALUES-3` 模块中的 CSS 值 & 单位。结合其他 CSS 模块，可能会扩展这些值类型的定义。
除了定义里的属性的特定值，本规范中定义的所有属性都接受 CSS 广义关键字作为属性值。处于可阅读性，他们不会被显式的重复出来

## 2 盒布局模式：display 属性

|          |                                                                                                                                         |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| 属性名   | display                                                                                                                                 |
| 值       | \[ \<display-outside> \| \<display-inside\> \] \| \<display-listitem\> \| \<display-internal\> \| \<display-box\> \| \<display-legacy\> |
| 初始值   | inline                                                                                                                                  |
| 应用于   | 所有元素                                                                                                                                |
| 可继承   | 不可                                                                                                                                    |
| 百分比   | n\/a                                                                                                                                    |
| 计算值   | 见具体规范的描述                                                                                                                        |
| 规范顺序 | 每个语法                                                                                                                                |
| 动画类型 | 不可作为动画                                                                                                                            |

_用户代理需要在所有媒体上支持这个属性，包括非视觉媒体_

display 属性定义了一个元素的显示类型，其由元素产生盒子方式的两个特性组成：

- 内部显示类型，（如果是非替换元素）定义了产生的格式上下文的类型，指明其后代盒子是如何排列的（替换元素的内部显示类型在 CSS 的作用范围之外）
- 外部显示内心，定义了主盒子自身是如何参与 flow 布局的

Text run 没有显示类型

一些 display 值可能会有额外的副作用：如 `list-item`，会产生一个 `::marker` 伪元素，而`none`则会让元素的整个子树移出盒子树。

