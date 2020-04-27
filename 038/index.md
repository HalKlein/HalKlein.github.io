# Shortcodes Tests


<!--more-->

{{< bilibili 47027633 >}}



## LoveIt shortcodes

**LoveIt** 在现有内置的 shortcodes 的基础上提供了多个 shortcodes.

### `style`

`style` shortcode 用来在你的文章中插入自定义样式.

`style` shortcode 使用两个参数. 第一个是自定义样式的内容, 第二个是包裹你要更改样式的内容的 HTML 标签, 默认值是 `p`.

一个 `style` 示例:

```markdown
{{< style "text-align: right" >}}
This is a right-aligned paragraph.
{{< /style >}}
```

呈现的输出效果如下:

{{< style "text-align: right" >}}
This is a right-aligned paragraph.
{{< /style >}}



### `image`

`image` shortcode 是 [`figure` shortcode](https://hugoloveit.com/zh-cn/theme-documentation-shortcodes/#figure) 的替代. `image` shortcode 可以充分利用 [lazysizes](https://github.com/aFarkas/lazysizes) 和 [lightgallery.js](https://github.com/sachinchoolur/lightgallery.js) 两个依赖库.

`image` shortcode 可以使用以下命名参数:

- **src**

  图片的 URL.

- **description**

  图片描述.

- **title**

  图片标题.

- **class**

  HTML `figure` 标签的 `class` 属性.

- **src_s**

  图片缩略图的 URL, 用在画廊模式中.

- **src_l**

  高清图片的 URL, 用在画廊模式中.

一个 `image` 示例:

{{< image src="https://i.loli.net/2019/12/25/bAyehHXWCwkLPSR.png" title="Lighthouse (`image`)" src-s="https://i.loli.net/2019/12/25/bAyehHXWCwkLPSR.png" src-l="https://i.loli.net/2019/12/25/bAyehHXWCwkLPSR.png" >}}



### `admonition`

`admonition` shortcode 支持 **12** 种 帮助你在页面中插入提示的横幅. 同时, `Markdown` 格式文本是支持的.

注意：note

摘要：abstract

信息：info

技巧：tip

成功：success

问题：question

警告：warning

失败：failure

危险：danger

Bug：bug

示例：example

引用：quote

`admonition` shortcode 可以使用以下命名参数:

- **type**

  `admonition` 横幅的类型, 默认值是 **note**

- **title**

  `admonition` 横幅的标题, 默认值是横幅的类型名称

- **details**

  如果设为 `true`, 横幅内容将是可展开/可折叠.

你还可以按 **type**, **title** 和 **details** 的顺序使用位置参数.

一个 `admonition` 示例:

{{< admonition type=bug title="横幅标题" details=false >}}
一个 **技巧** 横幅
{{< /admonition >}}
Or
{{< admonition tip "This is a tip" true >}}
一个 **技巧** 横幅
{{< /admonition >}}



### `mermaid`

[mermaid](https://mermaidjs.github.io/) 是一个可以帮助你在文章中生成图表和流程图的库, 类似 Markdown 的语法.

只需将你的 mermaid 代码插入 `mermaid` shortcode 中即可.

#### 流程图

一个 **流程图** `mermaid` 示例:

{{< mermaid >}}
graph LR;
    A[Hard edge] -->|Link text| B(Round edge)
    B --> C{Decision}
    C -->|One| D[Result one]
    C -->|Two| E[Result two]
{{< /mermaid >}}

#### 时序图

一个 **时序图** `mermaid` 示例:

{{< mermaid >}}
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail...
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!
{{< /mermaid >}}

#### 甘特图

一个 **甘特图** `mermaid` 示例:

{{< mermaid >}}
gantt
    dateFormat  YYYY-MM-DD
    title Adding GANTT diagram functionality to mermaid
    section A section
    Completed task            :done,    des1, 2014-01-06,2014-01-08
    Active task               :active,  des2, 2014-01-09, 3d
    Future task               :         des3, after des2, 5d
    Future task2               :         des4, after des3, 5d
    section Critical tasks
    Completed task in the critical line :crit, done, 2014-01-06,24h
    Implement parser and jison          :crit, done, after des1, 2d
    Create tests for parser             :crit, active, 3d
    Future task in critical line        :crit, 5d
    Create tests for renderer           :2d
    Add to mermaid                      :1d
{{< /mermaid >}}

#### 类图

一个 **类图** `mermaid` 示例:

{{< mermaid >}}
classDiagram
    Class01 <|-- AveryLongClass : Cool
    Class03 *-- Class04
    Class05 o-- Class06
    Class07 .. Class08
    Class09 --> C2 : Where am i?
    Class09 --* C3
    Class09 --|> Class07
    Class07 : equals()
    Class07 : Object[] elementData
    Class01 : size()
    Class01 : int chimp
    Class01 : int gorilla
    Class08 <--> C2: Cool label
{{< /mermaid >}}

#### 状态图

一个 **状态图** `mermaid` 示例:

{{< mermaid >}}
stateDiagram
    [*] --> Still
    Still --> [*]
    Still --> Moving
    Moving --> Still
    Moving --> Crash
    Crash --> [*]
{{< /mermaid >}}

#### Git 图

一个 **Git 图** `mermaid` 示例:

{{< mermaid >}}
gitGraph:
options
{
    "nodeSpacing": 100,
    "nodeRadius": 10
}
end
    commit
    branch newbranch
    checkout newbranch
    commit
    commit
    checkout master
    commit
    commit
    merge newbranch
{{< /mermaid >}}

#### 饼图

一个 **饼图** `mermaid` 示例:

{{< mermaid >}}
pie
    "Dogs" : 386
    "Cats" : 85
    "Rats" : 15
{{< /mermaid >}}



### `echarts`



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://1024.imfast.io/">https://1024.imfast.io/</a> </center>
