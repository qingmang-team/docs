# RAML 格式说明

<!-- MarkdownTOC depth=3 autolink="true" bracket="round" style="unordered" indent="  " autoanchor="false" -->

- [语言设计](#语言设计)
  - [语法介绍](#语法介绍)
- [标记说明](#标记说明)
  - [段落内容标记](#段落内容标记)
    - [id](#id)
    - [type](#type)
    - [image](#image)
    - [media](#media)
    - [table](#table)
  - [段落样式标记](#段落样式标记)
    - [blockquote](#blockquote)
    - [linetype](#linetype)
    - [align](#align)
  - [文本标记](#文本标记)
    - [start ＋ end](#start-＋-end)
    - [tag](#tag)
    - [width + height](#width--height)
    - [source](#source)
- [示例](#示例)
- [工具](#工具)

<!-- /MarkdownTOC -->

## 语言设计
`RAML (Ripple Article Markup Language)` 是用来描述文章正文的一种标记语言，其中包含文章的内容以及核心的结构，不同的端 Android/iOS/Web 都可以通过解释模版将其渲染成文章页面。

### 语法介绍
RAML 是基于 `JSON` 进行描述的，它会将文章正文转换成一个 `JSON List` 的字符串。其中 List 的每一个元素，都是`文章的一个段落`，每个段落，都会包含相关的正文内容和语法标记。

一个简单的，包含一段文字和一张图片的示例说明如下：
``` json
[
    {
      "id":"935d",
      "type":0,
      "text":"这里是文本类型的正文",

      "markups":[
        {
          "tag":"small",
          "start":4,
          "end":8,
          "source":"http://ifttt.com/medium",
          "width":0,
          "height":0
        }
      ]
    },
    {
      "id":"4231", 
      "type":1,
      "image": {
        "width":720,
        "height":480,
        "source":"http://ifttt.com/medium",
      }
    }
]
```

## 标记说明
RAML 中每一个 List 的元素，都是一个段落，其中包含一系列的 Key，这些 Key 可以分成如下几类：
* `段落内容标记`。每个段落都会有文本或者其它内容，它们会用特定的标记来表示；
* `段落样式标记`。每个段落也会有段落相关的样式标记，来表示它们在渲染时应该呈现的样子；
* `文本标记`。如果段落是文本，会包含一系列的文本标记，使得 RAML 不仅可以表示纯文本，还可以表示富文本内容。

### 段落内容标记

#### id
每个段落，都会有一个 id，是文章内对段落的标示。

#### type
每个段落，会有一个类型，它用来指明该段落的主体内容，不同的 type 其主要内容字段会有所不同。比如，如果是文本类型，内容存储在 text 标记中；如果是图片类型，则存储在 image 标记中。

目前支持的 type 包括：
* `0` 文本。内容在 text 中；
* `1` 单图。内容在 image 中；
* `2` 视频。内容在 media 中；
* `3` 音频。内容在 media 中；
* `4` 表格。内容在 table 中；
* `10` 空行符。无内容，视觉上为了增加段落间隔；
* `11` 分隔符。无内容，视觉上有一个分隔图标；

##### text
一个 `JSON Object`，富文本内容，包括：
* `text`。纯文本的 text 字段，段落中的文字内容；
* `markups`。辅助的标记信息，参看后面文本标记的介绍；

#### image
一个 `JSON Object`，包括：
* `source`。图片的 url 地址；
* `width`。图片的宽度，整形，单位是 px；
* `height`。图片的高度，整形，单位是 px；

#### media
一个 `JSON Object`，包括：
* `source`。视频的 url 地址；
* `cover`。视频的 cover 图片地址；

#### table
一个 `JSON Object`，包括：
* `rowCount`
* `colCount`

### 段落样式标记
在 text/image 等 `JSON Object` 中，还会有一些包含和段落样式相关的标记，包括如下这些。

#### blockquote
在 RAML 中，用 blockquote 统一表示`制表符`，包括了引用、项目制表，等等。它的取值包括：
* `0` 项目制表，这时候会出现 `li` 标签来表示制表符的类型，其中包括：
    - `type` 制表符类型，可以是 ol 或者是 ul
    - `level` 缩进层级，可以是 1、2，或者其他
    - `order` 次序，可以是 1、2，等等
* `1` 引用

#### linetype
在 RAML 中，用 linetype 表示一段文本的`语义`，取值包括：
* `h1` 文章标题
* `h2` 段落标题
* `h3` 二级段落标题
* `small` 字体缩小的文本
* `big` 字体放大的文本
* `pre` 类似代码之类的文本
* `aside` 类似辅助说明之类的文本，一般居中显示

特别说明，在 RAML 设计中，我们把对字体之类的控制放到段落级别，来体现不同区块的重要性差别。而不会放在文本标记中，避免同一段文本中样式过于复杂。

#### align
在 RAML，用 align 表示一个段落的排版位置，大部分时候，RAML 会避免使用这个标签而优先使用 linetype，仅会在一些比较复杂的样式里面使用 align 强行控制排版，它可能的取值包括：
* `left` 左对齐，默认对齐方式
* `center` 居中对齐

### 文本标记
当段落是文本时，可以添加一组的 markups 标签，来修饰某个部分文本所需的样式，以此来实现富文本效果。

#### start ＋ end
每个 markup 都有一组 start + end 来表示 markup 的作用范围，start 和 end 都是相对文本开头的偏移距离，取值范围是 `[0，段落长度）`，如果 start 缺失或者小于 0，视为 0；如果 end 缺失或者大于段落长度，可视为到段落末尾；如果 start 大于 end，视为 start 相等于 end。

#### tag
每个 markup 由一个 tag 标签来表示 markup 的类型，其命名取自于 html 规范。目前支持的 tag 包括：

| 标记   | 名字 | 备注                  |
| ------ | ---- | --------------------- |
| strong | 重点   | 表示所 markup 部分为重点      |
| em     | 强调   | 表示所 markup 部分需要强调     |
| a      | 链接   | 点击后跳转到其它地方的超链接        |
| img    | 图片   | 内嵌图片，会和文字在同一行（不需要环绕）  |
| sup    | 上标   | 右上角标                  |
| sub    | 下标   | 右下角标                  |

#### width + height
类似 tag 为 img 的 markup，可能会并没有覆盖的文字（start 等于 end），这时候，就需要通过 width 和 height 属性来表示其需要占用的范围，同样，width 和 height 也是整形表示 px。

#### source
类似 tag 为 img、a 的 markup，会有一些 web 地址信息，这时候，就需要通过 source 来存储对应的 web 地址。

## 示例
* [图文文章](./samples/wx_text_and_image.raml)。原文参见[这里](http://mp.weixin.qq.com/s?__biz=MzAwODIyMTUyNQ==&mid=2651025950&idx=2&sn=ced5e572286a44a75f3bf352cb2e7915&chksm=8085f424b7f27d3241bb7bb25eb44ce14eb24c53685a8d2cd84266f9e781d5d3d0f5cbc7ec28#rd)，包含图片、文字、引用等，用了非常多的样式信息，经过重新的排版转义后，生成样式更为统一的 RAML，渲染效果参见[这里](http://qingmang.me/magazines/7358/2017/02/12/7217265657588929268/)；
* [文本排版](./samples/wx_markup_and_li.raml)。原文参见[这里](http://mp.weixin.qq.com/s?__biz=MjM5MjYxNzY2NA==&mid=2650454589&idx=1&sn=b824e271ca187970e05bc14b5580f8fe&chksm=bead9f9289da1684645f13cd8725b4e23d5e672343602cb4d8530f8b1bf67c5626bc2da762fd#rd)，这是一篇包含很多文本样式的文章，转义后的效果参见[这里](http://qingmang.me/magazines/4880/2017/03/06/7917657200391947878/)；
* [视频和链接](./samples/wx_text_and_image.raml)。原文参见[这里](http://mp.weixin.qq.com/s?__biz=MjM5Mzc5NTk1OQ==&mid=2653005489&idx=1&sn=193dd71d6677073ffbe845b221c0196d&chksm=bd449ca98a3315bff0e405127b18ee4f19ccfedd95dde37b90f851901e193ecb34868ebf438c#rd)，有非常多的链接和视频，转义后的效果参见[这里](http://qingmang.me/magazines/7491/2017/03/06/3688605879307014948/)；
* [表格](./samples/table.raml)。原文参见[这里](http://m.pcauto.com.cn/x/957/9577254_all.html)，包含很多表格信息，转义后的效果参见[这里](http://qingmang.me/magazines/5678/-4443257712335137611/)；

## 工具
* [Android Demo](https://github.com/lianpian/raml-sdk-android)。在 Android 上实现 RAML 的解析和渲染；
