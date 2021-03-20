# 轻芒马克输出 RSS 开发者文档

该文档用于详述马克输出 RSS 相关话题，反馈建议可直接提 issue 或者 pr，或者加微信 hiqingmang 进开发者群参与讨论。

### 已经开发的应用合集

目前有以下一些应用，并持续更新中，感谢开发者们的贡献和分享；这里只是个列表，具体可看[这个文档](https://github.com/qingmang-team/docs/blob/master/content/note_rss_demo.md)，里边有更详细的说明。

- [大熊：博客主页的阅读动态](https://bearchao.com/)
- [钱争予：竖排中文 DEMO](https://realfish.github.io/anti-chronological-feed/)，[源码地址](https://github.com/realfish/anti-chronological-feed)
- [KyXu：将马克同步至 Slack](https://slack.com/apps/A0F81R7U7-rss)
- [囧哥：iOS 马克发送到 Flomo 捷径](https://www.icloud.com/shortcuts/e88a46a77df9430fb8339de742a8ffdb)
- [ERR_CONNECTION_RESET：自动同步到 flomo 和 Telegram bot](https://github.com/dake0805/Qingmang-mark)
- [extrastu：同步到 flomo 的 chrome 插件](https://www.notion.so/flomo-Plus-f440171cffbe40b997e6c45add04f658)
- [懒童一枚：将 RSS 转换为 Json 或者网页 div 模块](https://github.com/liutongl5/QMark-API)

### 结构执行 RSS 2.0 标准

参考 [RSS 2 specification](https://validator.w3.org/feed/docs/rss2.html)，可通过 [validator](https://validator.w3.org/feed/check.cgi) 校验

- 只输出最新的 30 条马克，这里兼顾了易用性和性能
- 马克按更新时间倒排序，这是与轻芒杂志 app 中的个人马克不同的地方，那里是按创建时间倒排的，但同一条马克的 [item guid tag](https://validator.w3.org/feed/docs/rss2.html#ltguidgtSubelementOfLtitemgt) 值固定，便于判断更新

### 马克 RSS url

地址如下，大部分用户的主页设置是不公开的，因此带有 secret 参数，用于校验权限，所以请妥善保管避免泄漏带来隐私问题： 

```
https://qingmang.me/users/{uid}/feed/<?secret=...>
```

少数用户本身的主页设置是公开的，那么就不带 secret，例如下边这个，可以随意使用: 

```
https://qingmang.me/users/11/feed/
```

### 马克正文的 html 结构

也就是 [item](https://validator.w3.org/feed/docs/rss2.html#hrelementsOfLtitemgt) description tag，其中正文是用 ![CDATA[ 包装起来的，为提升可读性，html 也是经过 prettify 的。

**马克引用的文章内容**

```html
<blockquote>
<!--    第一个 blockquote 是引用的文章内容，可能没有-->

    <!--    马克一段音频的情况，有音频地址，以及音频段对应的语音文本，还有对应的音频段时间点和总时长-->
    <audio controls="" src="https://chtbl.com/track/G1147/aphid.fireside.fm/d/1437767933/c7bd2b24-9119-4399-8943-ecdbdef8d646/36e12692-af29-475f-a956-f1e9cde53438.mp3">
    </audio>
    <p>
    <!--     该 tag 可选-->
    将近1300年前，经历4次失败的鉴真和尚，第五次东渡日本。
    </p>
    <cite>
    <!--    该 tag 可选-->
    [0:00:49]/[1:10:44]
    </cite>

    <!--    马克一段文本的情况-->
    <p>
    中国视协电视界职业道德建设委员会22日发布
    </p>

    <!--    马克多段文本的情况-->
    <p>
        中国视协电视界职业道德建设委员会22日发布
    </p>
    <p>
        电视艺术工作者应自觉追求德艺双修
    </p>

    <!--    马克一张图片的情况-->
    <figure>
        <img src="http://qiniuimg.qingmang.mobi/image/orion/c0ff048029ff19d86a1937b480a09edd_400_138.jpeg"/>
    </figure>

    <!--    马克多段文本和图片的情况-->
    <p>
        中国视协电视界职业道德建设委员会22日发布
    </p>
    <figure>
        <img src="http://qiniuimg.qingmang.mobi/image/orion/c0ff048029ff19d86a1937b480a09edd_400_138.jpeg"/>
    </figure>
    <p>
        电视艺术工作者应自觉追求德艺双修
    </p>
</blockquote>
```

**用户自己写的笔记内容**

```html
<aside>
    <!--    第二个 aside tag 是用户自己写的笔记内容，可能没有-->

    <!--    添加的笔记图片，一般在笔记内容的最前边，图片可选-->
    <figure>
        <img src="http://qiniuimg.qingmang.mobi/image/orion/c0ff048029ff19d86a1937b480a09edd_400_138.jpeg"/>
    </figure>
     一行笔记内容
    <br/>
    <!--    可选-->
    新的一行笔记内容
</aside>
```

**笔记更新时间和点亮的灯泡**

```html
<footer>
     <!--    footer 表示最后的 section，tag 一定会有-->

    <em>
    2021-03-05 18:56
    </em>
    &middot;
    <span>
    <!--      点亮的灯泡数，可选-->💡
    </span>
</footer>
```

### 马克正文图片防盗链问题

马克正文图片用的是文章里的原始链接，还请注意有些网站的图片防盗链问题，同时用户自己写的 annotation 里上传的图片不受防盗链的影响。

### 马克 web url 权限

也就是 [item guid tag](https://validator.w3.org/feed/docs/rss2.html#ltguidgtSubelementOfLtitemgt) 的值，地址如下：

```
https://qingmang.me/notes/{note_id}
```

由于涉及到权限问题，该地址目前只是用来遵循 rss 的协议表示全局唯一性，暂时没有开放访问。

### 其他

**还有其他开放的 api 吗？比如笔记增删改、马克到轻芒等等。**

暂时还未开放，但后期会提供给开发者的，包括用于开发第三方客户端的 api 等等。
