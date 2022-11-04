# 主题功能测试


<!--more-->

# 主题功能测试

## markdown 扩展语法

### KaTeX

$$c = \pm\sqrt{a^2 + b^2}$$

\\[ f(x)=\int_{-\infty}^{\infty} \hat{f}(\xi) e^{2 \pi i \xi x} d \xi \\]

\begin{equation*}
  \rho \frac{\mathrm{D} \mathbf{v}}{\mathrm{D} t}=\nabla \cdot \mathbb{P}+\rho \mathbf{f}
\end{equation*}

\begin{equation}
  \mathbf{E}=\sum_{i} \mathbf{E}\_{i}=\mathbf{E}\_{1}+\mathbf{E}\_{2}+\mathbf{E}_{3}+\cdots
\end{equation}

\begin{align}
  a&=b+c \\\\
  d+e&=f
\end{align}

\begin{alignat}{2}
   10&x+&3&y = 2 \\\\
   3&x+&13&y = 4
\end{alignat}

\begin{gather}
   a=b \\\\
   e=b+c
\end{gather}

\begin{CD}
   A @>a\>> B \\\\
@VbVV @AAcA \\\\
   C @= D
\end{CD}

### mhchem

$$\ce{CO2 + C -> 2 CO}$$

$$\ce{Hg^2+ ->[I-] HgI2 ->[I-] [Hg^{II}I4]^2-}$$

### 字符注音或注释

[Hugo]^(一个开源的静态网站生成工具)

### 分数

[浅色]/[深色]

[99]/[100]

### Font Awesome

去露营啦！:(fa-solid fa-campground fa-fw): 很快就回来。

真开心！:(fa-regular fa-grin-tears):

## 短代码

保持 `markdown` 的整洁，避免嵌入过多 `html`。

### 1 figure

{{< figure src="/images/lighthouse.jpeg" title="Lighthouse (figure)" >}}

### 2 gist

{{< gist amcones 38f77b13cc454c3170cbcb632732fbf3 >}}

### 3 highlight

{{< highlight html >}}
<section id="main">
    <div>
        <h1 id="title">{{ .Title }}</h1>
        {{ range .Pages }}
            {{ .Render "summary"}}
        {{ end }}
    </div>
</section>
{{< /highlight >}}

### 4 param

{{< param description >}}

### 5 ref和relf

### 6 tweet

{{< tweet user="SanDiegoZoo" id="1453110110599868418" >}}

### 7 viemo

{{< vimeo 146022717 >}}

### 8 youtube

{{< youtube w7Ft2ymGmfc >}}

## 扩展短代码

### 1 style

{{< style "text-align:right;" >}}
This is a **right-aligned** paragraph.
{{< /style >}}

### 2 link

{{< version 0.2.0 >}}

`link` shortcode 是 [Markdown 链接语法](../basic-markdown-syntax#links) 的替代。
`link` shortcode 可以提供一些其它的功能并且可以在代码块中使用。

{{< version 0.2.10 >}} 支持 [本地资源引用](../theme-documentation-content#contents-organization) 的完整用法。

`link` shortcode 有以下命名参数：

* **href** *[必需]*（**第一个**位置参数）

    链接的目标。

* **content** *[可选]*（**第二个**位置参数）

    链接的内容，默认值是 **href** 参数的值。

    *支持 Markdown 或者 HTML 格式。*

* **title** *[可选]*（**第三个**位置参数）

    HTML `a` 标签 的 `title` 属性，当悬停在链接上会显示的提示。

* **card** *[可选]*（**第四个**位置参数）{{< version 0.2.12 >}}

    是否显示为卡片式链接，默认值 `false`。

* **download** *[可选]* {{< version 0.2.12 >}}

    HTML `a` 标签 的 `download` 属性。

* **class** *[可选]*

    HTML `a` 标签 的 `class` 属性。

* **rel** *[可选]*

    HTML `a` 标签 的 `rel` 补充属性。

* **external-icon** *[可选]* {{< version 0.2.14 >}}

    是否自动显示外链图标。

* **noreferrer** *[可选]* {{< version 0.2.16 >}}

    `rel` 属性是否添加 `noreferrer`, 默认：`true`。

一个 `link` 示例：

```go-html-template
{{</* link "https://assemble.io" */>}}
或者
{{</* link href="https://assemble.io" */>}}

{{</* link "mailto:contact@revolunet.com" */>}}
或者
{{</* link href="mailto:contact@revolunet.com" */>}}

{{</* link "https://assemble.io" Assemble */>}}
或者
{{</* link href="https://assemble.io" content=Assemble */>}}
```

呈现的输出效果如下：

* {{< link "https://assemble.io" >}}
* {{< link "mailto:contact@revolunet.com" >}}
* {{< link "https://assemble.io" Assemble >}}

一个带有标题的 `link` 示例：

```go-html-template
{{</* link "https://github.com/upstage/" Upstage "Visit Upstage!" */>}}
或者
{{</* link href="https://github.com/upstage/" content=Upstage title="Visit Upstage!" */>}}
```

呈现的输出效果如下 （将鼠标悬停在链接上，会有一行提示）:

{{< link "https://github.com/upstage/" Upstage "Visit Upstage!" >}}

一个卡片式 `link` 示例：

```go-html-template
{{</* link "https://github.com/hugo-fixit/FixIt" "FixIt Theme" "source of FixIt Theme" true */>}}
或者
{{</* link href="https://github.com/hugo-fixit/FixIt" content="FixIt Theme" title="source of FixIt Theme" card=true */>}}
```

呈现的输出效果如下：

{{< link "https://github.com/hugo-fixit/FixIt" "FixIt Theme" "source of FixIt Theme" true >}}

一个可下载的 `link` 示例：

```go-html-template
{{</* link href="/music/Wavelength.mp3" content="Wavelength" title="Download Wavelength.mp3" download="Wavelength.mp3" */>}}

{{</* link href="/music/Wavelength.mp3" content="Wavelength.mp3" title="Download Wavelength.mp3" download="Wavelength.mp3" card=true */>}}
```

呈现的输出效果如下：

{{< link href="/music/Wavelength.mp3" content="Wavelength.mp3" title="Download Wavelength.mp3" download="Wavelength.mp3" >}}

{{< link href="/music/Wavelength.mp3" content="Wavelength.mp3" title="Download Wavelength.mp3" download="Wavelength.mp3" card=true >}}

### 3 image

### 4 admonition

{{< admonition >}} 一个 **注意** 横幅 {{< /admonition >}}

{{< admonition abstract >}} 一个 **摘要** 横幅 {{< /admonition >}}

{{< admonition info >}} 一个 **信息** 横幅 {{< /admonition >}}

{{< admonition tip >}} 一个 **技巧** 横幅 {{< /admonition >}}

{{< admonition success >}} 一个 **成功** 横幅 {{< /admonition >}}

{{< admonition question >}} 一个 **问题** 横幅 {{< /admonition >}}

{{< admonition warning >}} 一个 **警告** 横幅 {{< /admonition >}}

{{< admonition failure >}} 一个 **失败** 横幅 {{< /admonition >}}

{{< admonition danger >}} 一个 **危险** 横幅 {{< /admonition >}}

{{< admonition bug >}} 一个 **Bug** 横幅 {{< /admonition >}}

{{< admonition example >}} 一个 **示例** 横幅 {{< /admonition >}}

{{< admonition quote >}} 一个 **引用** 横幅 {{< /admonition >}}

### 5 mermaid

### 6 echarts

### 7 mapbox

### 8 music

{{< music auto="https://music.163.com/#/playlist?id=874502722" >}}

### 9 bilibili

{{< bilibili BV1Kt4y1K7tj >}}

### 10 typeit

{{< typeit >}}
这是一个带有基于 [TypeIt](https://typeitjs.com/) 的 **打字动画** 的 *段落* ...
{{< /typeit >}}
{{< typeit code=java >}}
public class HelloWorld {
    public static void main(String []args) {
        System.out.println("Hello World");
    }
}
{{< /typeit >}}
{{< typeit group=paragraph >}}
**首先**, 这个段落开始
{{< /typeit >}}

{{< typeit group=paragraph >}}
**然后**, 这个段落开始
{{< /typeit >}}

### 11 script

{{< script >}}
console.log('Hello FixIt!');
{{< /script >}}

### 12 details

{{< details "**Copyright** 2022." >}}
*All pages and graphics on this web site are the property of FixIt.*
{{< /details >}}
或者
{{< details summary="**Copyright** 2022." >}}
*All pages and graphics on this web site are the property of FixIt.*
{{< /details >}}

### 13 center-quote

{{< center-quote >}}
**hello** *world*  
this is a center-quote shortcode example.
{{< /center-quote >}}

### 14 fixit-encryptor

### 15 raw

{{< raw >}}行内公式：\(\mathbf{E}=\sum_{i} \mathbf{E}*{i}=\mathbf{E}*{1}+\mathbf{E}*{2}+\mathbf{E}*{3}+\cdots\){{< /raw >}}

{{< raw >}}
公式块：
\[ a=b+c \\ d+e=f \]
{{< /raw >}}

原始的带有 Markdown 和 HTML 语法的内容：{{< raw "span" >}}**Hello** <strong>FixIt</strong>{{< /raw >}}

### 16 reward


---

> 作者: [amcones](https://amcones.cn)  
> URL: https://amcones.github.io/%E4%B8%BB%E9%A2%98%E5%8A%9F%E8%83%BD%E6%B5%8B%E8%AF%95/  

