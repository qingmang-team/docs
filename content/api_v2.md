# 轻芒内容 APIs V2

<!-- MarkdownTOC depth=2 autolink="true" bracket="round" style="unordered" indent="  " autoanchor="false" -->

- [APIs 设计](#apis-设计)
  - [返回数据](#返回数据)
  - [错误处理](#错误处理)
  - [测试方式](#测试方式)
- [应用（App）](#应用（app）)
  - [app.auth](#appauth)
  - [app.get](#appget)
- [分类（Category）](#分类（category）)
  - [category.list](#categorylist)
  - [category.get](#categoryget)
- [文章（Article）](#文章（article）)
  - [article.list](#articlelist)
  - [article.dump](#articledump)
  - [article.search](#articlesearch)
  - [article.get](#articleget)

<!-- /MarkdownTOC -->

## APIs 设计
轻芒 APIs 在域名 `https://api.qingmang.me/v2/` 下，它遵循如下的一些设计理念：
* APIs 的接口命名遵循 `RPC 风格`，形如 `域名/主体.操作`，主体和操作都使用小写开头的驼峰命名法，比如：获取文章内容的主体是 `article`，操作是 `get`，它的完整 API 是 `https://api.qingmang.me/v2/article.get`；
* APIs 支持 `Get` 或 `Post` 请求，所需的信息都放在参数中（或者是 Form 中），参数名均为小写加下划线；
* APIs 始终会返回 `Http Code 200`（除非服务挂了），以及对应的 Json Object，其中包含了所需的数据或者错误信息；

### 返回数据
APIs 都会返回一个 Json Object，它的 Code Style 沿用自 `Google Json Style Guide` ([中文](https://github.com/darcyliu/google-styleguide/blob/master/JSONStyleGuide.md) | [原版](https://google.github.io/styleguide/jsoncstyleguide.xml))。

重点部分如下：
* 属性的定义采取驼峰结构，形如 `theSpecialProperty`，一般是小写字母开头；
* 保持 Json 结构相对扁平，除非是非常明确的数据结构；
* 如果属性值是列表，采取 `复数` 形态来命名属性名，比如 `thumbnails`，而如果需要表示一个计数，最好是 count 结尾，形如 `itemCount`；
* 如果一个属性是枚举类别，最好使用字符串来描述，而不要转义成一个 int；
* 如果为 null，除非有特别涵义，否则不返回；
* 如果用字符串来表示时间、间隔、经纬度等等，需要遵循对应的 RFC 或 ISO 规范；

返回的 Json Object 中，为了规范相近语意的内容在不同 Json 中保持定义统一，规定了一系列的保留字，这些保留字我们不遵循 Google 的定义，而是采取自己的规范，示例如下：
```json
{
  "ok": true,
  "error": {
      "type": "invalid_token",
      "desc": "用户 token 已经失效，可能是超时或者登出"
  },
  "nextUrl": "http://url_to_next_page/if_has_more",
  "hasMore": true
}
```

其中：
* `ok` 表示请求是否成功，`true` 为成功，`false` 为请求错误；
* `error` 表示具体的错误信息，仅当 ok 为 false 的时候生效，它包括错误类型 `type` 和错误的具体描述 `desc`，可以帮助了解错误的具体原因；
* `hasMore` 表示是否还有下一页，当返回结果是列表的时候生效，`true` 为还有下一页，`false` 表示没有下一页了；
* `nextUrl` 表示下一页的 url 地址，当 hasMore 为 true 的时候生效；

此外，如果请求成功，还会包括具体的返回数据，其定义可以参见具体 APIs 的介绍。

### 错误处理
当请求失败，返回 Json 中会包含 `error` 错误信息，其中 `type` 的定义可以参见下表：

| 错误类型 | 含义和处理方式 |
|:--------|:-------------|
| invalid_token | 错误的 token 信息，无法通过校验，可能需要重新进行授权 |
| miss_parameter | 缺少必要的参数信息 |
| no_permission | 缺少足够的权限来完成对应的操作 |
| unknown_error | 其它的位置错误，描述中会包含具体的信息 |

### 测试方式

## 应用（App）
每个合作伙伴，在轻芒都是定义成一个应用（App），轻芒会为每个通过审核的 App 分配对应的访问权限和可使用的 Quota。

一个 App 会通过如下的 Json Object 进行表示：
```json
{
  "appId": "assigned_app_id",
  "name": "合作伙伴的名字",
  "icon": "http://a_icon_for_partener",
  "desc": "合作伙伴的描述",

  "status": "active"
}
```

其中：
* `appId` 表示对应的 App ID，审核通过后会自动分配，而 `name` `icon` `desc` 则为其它提交的基本信息；
* `status` 表示应用的审核状态，`active` 表示审核通过，`inactive` 表示暂不可用；

### app.auth
授权接口，在访问任何其他接口前，合作伙伴都需要通过该接口来获得 `token`，拿到 `token` 后需要自行持久化便于后续访问其它 APIs 时鉴权使用。

App 审核通过后，会分配一个 `App ID` 和一个 `Secret Key`，其中：
* `App ID` 用来标示应用，为了控制服务端压力，每个 App 会控制一定的访问频次，App ID 一旦分配就不会再变更；
* `Secret Key` 用来计算签名，Secret Key 是可以更新的，一旦更新，基于原来的 Key 计算的 Token 就会全部立刻失效，以此，来防止 Key 被盗用导致的问题；

*该 APIs 仅可在服务端调用，避免泄漏 Secret Key*

#### 参数

| 参数 | 类型 | 是否必须 | 示例 | 其它说明 |
|:--|:--|:--|:--|:--|
| app_id | string | 是 | qingmang | 分配给第三方的 App ID |
| sign | string | 是 | a12ce7f | 根据算法，基于 Secret Key 计算而来的签名信息 |
| ts | long | 是 | 1491038197 | 当前的时间戳 |

其中，`sign` 的具体算法是：
```
sign = hmac_sha1(Secret Key, App ID + ':' + Timestamp)
```

一段用来签名的 `Python` 代码如下：
```python
#!/usr/bin/env python

import sys
from hashlib import sha1
import hmac

if __name__ == "__main__":
    key = sys.argv[1]
    appid = sys.argv[2]
    ts = sys.argv[3]
    msg = "%s:%s" % (appid, ts)
    sign = hmac.new(key, msg, sha1).digest()
    print sign.encode("hex")
```

可以传入签名和时间戳来计算 `sign`:
```
python hmac-sha1.py secret-key app-id 1491038197
5fc145cab4285f0a29d3c35533d32db1a957b125
```

#### 返回
```json
{
  "ok": true,
  "app": {
    "appId": "qingmang",
    "name": "轻芒杂志",
    "icon": "http://static.wdjimg.com/rippleweb/images/logo.7b1996ef.png",
    "desc": "全新的、全面美好的兴趣杂志",

    "status": "active"
  },
  "token": "abc1234sxba"
}
```

请求成功后，不仅会返回 `app` 的信息，更包括了 `token` 信息，保存它，可以用来后续 APIs 的校验。

### app.get
获得 App 信息的接口，也可以用做校验 `token` 是否有效。

#### 参数

| 参数 | 类型 | 是否必须 | 示例 | 其它说明 |
|:--|:--|:--|:--|:--|
| token | string | 是 | abc1234sxba | 从 `app.auth` 中获得的 token 信息 |

#### 返回
```json
{
  "ok": true,
  "app": {
    "appId": "qingmang",
    "name": "轻芒杂志",
    "icon": "http://static.wdjimg.com/rippleweb/images/logo.7b1996ef.png",
    "desc": "全新的、全面美好的兴趣杂志",

    "status": "active"
  }
}
```

请求成功后，会返回 `app` 的信息。

## 分类（Category）
轻芒可以通过不同纬度的分类（Category）来提供文章内容，常见的维度包括：
* `兴趣`。轻芒将内容分成数百个不同的兴趣，比如：`家居`，`科技`，`咖啡`，`旅行`，等等，可以通过不同兴趣来获取对应的文章；
* `应用`。轻芒收录了上千个应用和公众号，比如：`少数派`，`好奇心日报`，`清单`，等等，可以通过不同应用来获取对应的文章；
* `类别`。轻芒也支持通过不同的内容类比来获取相应的文章，比如：`文字`，`图片`，`视频`，等等；

此外，如果有其它定制化的需求，比如，需要某个兴趣下面的视频内容，或者是特定几个应用的内容，轻芒也可以特殊定制来实现。

一个分类的信息，会通过如下的 Json Object 来表示：
```json
{
  "categoryId": "assigned_category_id",
  "type": "interest",
  "name": "分类名",
  "icon": "http://a_icon_for_partener",
  "description": "分类的描述",

  "subCategories": [{
    "categoryId": "p50",
    "title": "子分类的名称"
  }]
}
```

其中：
* `categoryId` 是该分类的 id，`type` 是该分类的类型，包括：`interest`，`app`，`content`，等。此外，还包含 `name`，`icon`，等基本信息（可能空）；
* 在部分分类下，还可能包含更具体的 `subCategories`，说明这个分类下面还有子分类，可以用子分类来获取它对应的 Timeline 信息，比如，一个应用就可能包含多个不同频道的内容；

### category.list
获得提供给该合作伙伴的全部分类列表。

#### 参数

| 参数 | 类型 | 是否必须 | 示例 | 其它说明 |
|:--|:--|:--|:--|:--|
| token | string | 是 | abc1234sxba | 从 `app.auth` 中获得的 token 信息 |

#### 返回
```json
{
  "ok": true,
  "categories": [{
    "categoryId": "assigned_category_id",
    "type": "interest",
    "icon": "http://a_icon_for_partener",
    "description": "分类的描述",
  }],
  "hasMore": false
}
```

请求成功后，会返回 `categories` 的列表。

### category.get
获得具体 Category 的详细信息。

#### 参数

| 参数 | 类型 | 是否必须 | 示例 | 其它说明 |
|:--|:--|:--|:--|:--|
| token | string | 是 | abc1234sxba | 从 `app.auth` 中获得的 token 信息 |
| category_id | string | 是 | i1234 | 已知的特定分类的 ID 信息 |

#### 返回
```json
{
  "ok": true,
  "category": {
    "categoryId": "assigned_category_id",
    "type": "interest",
    "icon": "http://a_icon_for_partener",
    "description": "分类的描述",
  },
  "hasMore": false
}
```

请求成功后，会返回对应的 `category`。

## 文章（Article）
文章（Article）指的是一个页面的正文内容，轻芒提供 APIs，将源站中提取出结构化的信息。

一个文章，会用如下的 Json Object 来表示：
```json
{
  "articleId": -8379746561699078860,

  "title": "三星Galaxy S8/S8+上手体验",
  "snippet": "转眼又到2017年的春季，S系列终于有机会上头条，Galaxy S8有没有一雪前耻，更进一步呢？这是我们将要探究的内容。",
  "author": "科技美学",
  "publishTimestamp": 1490803200000,
  "crawlerTimestamp": 1490893250880,
  "covers": [{
    "url": "http://qiniuimg.qingmang.mobi/image/orion/d534586e7e0f28f2deb1bcda253f9e1a_1920_1080.jpeg",
    "height": 1080,
    "width": 1920
  }],
  "images": [{
    "url": "http://link_to_image",
    "width": 1024,
    "height": 2048
  }],
  "videos": [{
    "url": "http://api.qingmang.me/v1/video.redirect?url=https://v.qq.com/iframe/preview.html?vid%3Di0388m50vls%26width%3D500%26height%3D375%26auto%3D0",
    "duration": 501.12,
    "width": 1920,
    "height": 1072
  }],
  "musics": [{
    "url": "http://res.wx.qq.com/voice/getvoice?mediaid=MjM5ODQwNDQxNF8yNjUwNjk2MTIx",
    "name": "2017.03.30"
  }],
  "tags": ["美食"],
  "templateType": "text",
  "categories": [{
    "categoryId": "p50",
    "title": "科技美学"
  }],
  "contentUrl": "http://qingmang.me/articles/-8379746561699078860",

  "webUrl": "文章的原文链接",
  "appUrl": "文章在应用中打开的链接",
  "providerName": "科技美学",
  "providerIcon": "http://img.wdjimg.com/image/orion/e7b233d4c4d93c9c5411429d1b66a7cd_292_292.jpeg",
  "providerPackageName": "org.wandoujia.mp.kejimx",

  "contentFormat": "raml",
  "content": "正文内容",
  "keywords": [{
    "word": "s8",
    "score": 100
  }]
}
```

其中，可以分成三部分的内容：
* `轻芒抽取出来的结构化内容`。根据源站本身的内容，和轻芒对内容的理解，将结构化的信息提取出来，包括了文章的唯一标示，标题，描述，作者，发布时间，头图，文章中包含的图片、视频，关联的频道、Tag、兴趣，等等；
* `原文的基本信息`。原文本身的一些信息，包括原文链接 `webUrl`，源站的名字、包名，等等；
* `轻芒抽取的正文内容`。轻芒除了会抽取文章的基本信息，还会抽取其正文信息进行计算和排版，包括了正文的内容 `content` 和对应的排版格式 `contentFormat`，以及从正文抽取出来的关键字 `keywords` 信息。由于这部分内容大小比较大，大部分接口中不提供，仅可通过 `article.get` 接口来获取；

其中，`templateType` 是轻芒推算的文章适合的展示样式，包括：

| 版式样式 | 含义 |
|:--------|:--- |
| text | 一般的长文，默认样式 |
| short_text | 短文本 |
| video | 视频 |
| image | 单图 |
| galary | 多图 |

### article.list
获得给定分类下的文章列表。

#### 参数

| 参数 | 类型 | 是否必须 | 示例 | 其它说明 |
|:--|:--|:--|:--|:--|
| token | string | 是 | abc1234sxba | 从 `app.auth` 中获得的 token 信息 |
| category_id | string | 是 | i1567 | 从 `category.list` 中获取的 `categoryId` 信息，或者 `subCategories` 中的 ID 信息 |

#### 返回
```json
{
  "ok": true,
  "articles": [{
    "title": "三星Galaxy S8/S8+上手体验",
    "snippet": "转眼又到2017年的春季，S系列终于有机会上头条，Galaxy S8有没有一雪前耻，更进一步呢？这是我们将要探究的内容。",
    "author": "科技美学",
    "publishTimestamp": 1490803200000,
    "crawlerTimestamp": 1490893250880,
    "covers": [{
      "url": "http://qiniuimg.qingmang.mobi/image/orion/d534586e7e0f28f2deb1bcda253f9e1a_1920_1080.jpeg",
      "height": 1080,
      "width": 1920
    }],
    "images": [{
      "url": "http://link_to_image",
      "width": 1024,
      "height": 2048
    }],
    "videos": [{
      "url": "http://api.qingmang.me/v1/video.redirect?url=https://v.qq.com/iframe/preview.html?vid%3Di0388m50vls%26width%3D500%26height%3D375%26auto%3D0",
      "duration": 501.12,
      "width": 1920,
      "height": 1072
    }],
    "musics": [{
      "url": "http://res.wx.qq.com/voice/getvoice?mediaid=MjM5ODQwNDQxNF8yNjUwNjk2MTIx",
      "name": "2017.03.30"
    }],
    "tags": ["美食"],
    "templateType": "text",
    "categories": [{
      "categoryId": "p50",
      "title": "科技美学"
    }],
    "contentUrl": "http://qingmang.me/articles/-8379746561699078860",

    "webUrl": "文章的原文链接",
    "appUrl": "文章在应用中打开的链接",
    "providerName": "科技美学",
    "providerIcon": "http://img.wdjimg.com/image/orion/e7b233d4c4d93c9c5411429d1b66a7cd_292_292.jpeg",
    "providerPackageName": "org.wandoujia.mp.kejimx",
  }],
  "hasMore": false
}
```

返回的 `articles` 中，包含除了正文相关信息的其它数据。

### article.dump
和 `article.list` 类似，也是获得给定分类下的文章列表。但所不同的是，只要返回过 的文章，就不再会重新再提供了，进入分类的文章仅仅会被取走一次。

*该接口需要在服务端调用，避免多次调用导致返回内容丢失*

#### 参数

| 参数 | 类型 | 是否必须 | 示例 | 其它说明 |
|:--|:--|:--|:--|:--|
| token | string | 是 | abc1234sxba | 从 `app.auth` 中获得的 token 信息 |
| category | string | 是 | i1567 | 从 `category.list` 中获取的 `categoryId` 信息 |
| reset | boolean | 否，默认为 false | true | 是否重置全部已读信息，如果需要从头再取，则需要将该参数设置为 true |

#### 返回
```json
{
  "ok": true,
  "articles": [{
    "title": "三星Galaxy S8/S8+上手体验",
    "snippet": "转眼又到2017年的春季，S系列终于有机会上头条，Galaxy S8有没有一雪前耻，更进一步呢？这是我们将要探究的内容。",
    "author": "科技美学",
    "publishTimestamp": 1490803200000,
    "crawlerTimestamp": 1490893250880,
    "covers": [{
      "url": "http://qiniuimg.qingmang.mobi/image/orion/d534586e7e0f28f2deb1bcda253f9e1a_1920_1080.jpeg",
      "height": 1080,
      "width": 1920
    }],
    "images": [{
      "url": "http://link_to_image",
      "width": 1024,
      "height": 2048
    }],
    "videos": [{
      "url": "http://api.qingmang.me/v1/video.redirect?url=https://v.qq.com/iframe/preview.html?vid%3Di0388m50vls%26width%3D500%26height%3D375%26auto%3D0",
      "duration": 501.12,
      "width": 1920,
      "height": 1072
    }],
    "musics": [{
      "url": "http://res.wx.qq.com/voice/getvoice?mediaid=MjM5ODQwNDQxNF8yNjUwNjk2MTIx",
      "name": "2017.03.30"
    }],
    "tags": ["美食"],
    "templateType": "text",
    "categories": [{
      "categoryId": "p50",
      "title": "科技美学"
    }],
    "contentUrl": "http://qingmang.me/articles/-8379746561699078860",

    "webUrl": "文章的原文链接",
    "appUrl": "文章在应用中打开的链接",
    "providerName": "科技美学",
    "providerIcon": "http://img.wdjimg.com/image/orion/e7b233d4c4d93c9c5411429d1b66a7cd_292_292.jpeg",
    "providerPackageName": "org.wandoujia.mp.kejimx",
  }],
  "hasMore": false
}
```

同上，返回的 `articles` 中，包含除了正文相关信息的其它数据。

### article.search
通过关键字，搜索相关的文章。

#### 参数

| 参数 | 类型 | 是否必须 | 示例 | 其它说明 |
|:--|:--|:--|:--|:--|
| token | string | 是 | abc1234sxba | 从 `app.auth` 中获得的 token 信息 |
| query | string | 是 | Google | 搜索的关键词 |
| category | string | 否 | i1567 | 从 `category.list` 中获取的 `categoryId` 信息，如果添加该参数，则从该分类下进行搜索 |
| sort | string | 否，默认为相关度（relevant） | time | 搜索结果返回的方式，支持 `relevant` 按照相关度从高到低返回，`time` 按照时间从新到旧返回 |

#### 返回
```json
{
  "ok": true,
  "articles": [{
    "articleId": 251,

    "title": "这是文章的标题",
    "desc": "这是文章的描述信息",
    "snippet": "这是文章的摘要信息",
    "author": "文章的作者",
    "publishTimestamp": 140000000000,
    "crawlerTimestamp": 140000000000
  }],
  "hasMore": false
}
```

同上，返回的 `articles` 中，包含除了正文相关信息的其它数据。

### article.get
获得给定的文章，包含文章正文。

#### 参数

| 参数 | 类型 | 是否必须 | 示例 | 其它说明 |
|:--|:--|:--|:--|:--|
| token | string | 是 | abc1234sxba | 从 `app.auth` 中获得的 token 信息 |
| id | string | 是 | 12345 | 文章的 id，即 article 中的 `articleId` |
| format | string | 否，默认为 html（html) | raml | 文章正文的格式，支持 `html`, `[raml](../raml/intro.md)` |
| need_keywords | boolean | 否，默认为 false | true | 是否需要计算正文的关键字，默认为 false |

#### 返回
```json
{
  "ok": true,
  "article": {
    "articleId": -8379746561699078860,

    "title": "三星Galaxy S8/S8+上手体验",
    "snippet": "转眼又到2017年的春季，S系列终于有机会上头条，Galaxy S8有没有一雪前耻，更进一步呢？这是我们将要探究的内容。",
    "author": "科技美学",
    "publishTimestamp": 1490803200000,
    "crawlerTimestamp": 1490893250880,
    "covers": [{
      "url": "http://qiniuimg.qingmang.mobi/image/orion/d534586e7e0f28f2deb1bcda253f9e1a_1920_1080.jpeg",
      "height": 1080,
      "width": 1920
    }],
    "images": [{
      "url": "http://link_to_image",
      "width": 1024,
      "height": 2048
    }],
    "videos": [{
      "url": "http://api.qingmang.me/v1/video.redirect?url=https://v.qq.com/iframe/preview.html?vid%3Di0388m50vls%26width%3D500%26height%3D375%26auto%3D0",
      "duration": 501.12,
      "width": 1920,
      "height": 1072
    }],
    "musics": [{
      "url": "http://res.wx.qq.com/voice/getvoice?mediaid=MjM5ODQwNDQxNF8yNjUwNjk2MTIx",
      "name": "2017.03.30"
    }],
    "tags": ["美食"],
    "templateType": "text",
    "categories": [{
      "categoryId": "p50",
      "title": "科技美学"
    }],
    "contentUrl": "http://qingmang.me/articles/-8379746561699078860",

    "webUrl": "文章的原文链接",
    "appUrl": "文章在应用中打开的链接",
    "providerName": "科技美学",
    "providerIcon": "http://img.wdjimg.com/image/orion/e7b233d4c4d93c9c5411429d1b66a7cd_292_292.jpeg",
    "providerPackageName": "org.wandoujia.mp.kejimx",

    "contentFormat": "raml",
    "content": "正文内容",
    "keywords": [{
      "word": "s8",
      "score": 100
    }]
  }
}
```

请求成功后，会返回包含正文信息的 `article`。
