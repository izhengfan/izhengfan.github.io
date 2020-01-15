---
layout: cnpost
title:  字体概谈
date: 2015-08-15 10:00:51 
categories: cn
tags: type web
---

__目录__

* content
{:toc}

字体排印（typography）和字体（type）是很宽的概念，本文主要讨论网页设计中的字体选择问题，故借用 CSS 中 字族（font family）的概念。

### 衬线和无衬线

![lXHpTg.jpg](https://s2.ax1x.com/2020/01/15/lXHpTg.jpg)

衬线（serif）是个重要概念。看上图，衬线作为字体笔画末端的修饰性线条，设计风格多样，能突显某种风格和个性，提升可读性。使用衬线的字体称为衬线字体（Serif）。同理，不使用衬线的字体称为无衬线字体（Sans-serif）。CSS 的字体概念中，衬线和无衬线是两大字族组（generic family）。第三大字族组是等宽字体（Monospace），等宽字体的每个字符占用相同的宽度。

<!-- ![](/images/font-family.png) -->
![lXHifs.png](https://s2.ax1x.com/2020/01/15/lXHifs.png)

不同字族组各有适用场合。通常，衬线字体适用于印刷品，千变万化的衬线设计带来风格多样的视觉效果，传达给读者更具功能性的、更微妙的感受。在数码显示屏上，由于所有形状都是由像素点拼成，字符显示效果就受限于屏幕分辨率。尤其在低分屏上，衬线字体往往边沿锯齿严重，或者线条过于纤细以致影响辨识——中文字体尤为严重。所以像 Arial 或微软雅黑这样的无衬线字体，笔画粗壮，更易辨识，是更佳选择，故在网页设计中被广泛使用。等宽字体则多作为编程字体。

### 易辨识性和可读性

易辨识性（legibility）指的是一个字符被辨识的难易程度。典型例子是大写字母 I、小写字母 l 和数字 1 的区分，看下图：

<!-- ![](/images/font-legi.png) -->
![lXHPYj.png](https://s2.ax1x.com/2020/01/15/lXHPYj.png)

可以看到，诸如 Consolas 这样的字体，三者很容易区分。某些字体就不行，比如 Arial 中的大写字母 I 和小写字母 l，Times New Roman 的小写字母 l 和数字 1。

对于字符准确度有很高要求的场合，比如编程、密码显示，在选择字体时，易辨识性需要着重考虑。

可读性（readability）是更宽的概念，不仅受字体设计，也受字体排印中的各种其他因素的影响。如果排版糟糕，即使选择了合适的字体，显示质量也不会高。

### 跨平台

网页字体选择一般原则：多用无衬线，少用衬线。不过，不同操作系统有不同的字体渲染策略。例如，Mac 倾向于还原字体原始设计，在高分屏上显示优美，在低分屏上则往往字体过于纤弱影响可读性。Windows 反其道而行之，把易辨识度放在首位，字体渲染上常使用很强的 hinting（抗锯齿微调），有点「重构」字形的意思，以确保文字在任何素质的屏幕上都可读，副作用则是在高分屏上不如 Mac 优雅。此外，不同操作系统内置字体也不尽相同，会带来不同的显示效果。

常用操作系统有 Windows、Mac、Linux，这里也只考虑这三者。在本站中，字体由下面的 CSS 代码指定：

```css
body {
    font-family: "Georgia","Hiragino Sans GB", "Droid Sans Fallback", "Microsoft JhengHei", "Microsoft YaHei",sans-serif;
}
```

CSS 会选择 `font-family` 列表中的第一个字体，如果第一个字体在当前系统里不存在或者对于当前语言不适用，则选择后面的字体。由于通常拉丁字体不包括中文，而中文字体则往往包含拉丁字符，所以建议拉丁字体声明放在前，中文字体声明放在后。在本站中，选择 Georgia 作为英文字体。这款衬线字体设计精良，风格优雅，同时不失可辨识度和可读性。中文字体情况更复杂，因为不同操作系统内置的中文字体不同，Hiragino Sans GB （冬青黑体）用于 Mac，Droid Sans Fallback 用于 Linux，两者都是设计和口碑较好的无衬线字体。Yahei （雅黑）和 Jhenghei（正黑）都常用于 Windows。之所以把 Mac 和 Linux 字体声明放在 Windows 前面，是因为 Mac 可能因为装了 Office 套装而含有 Yahei 等微软字体，而 Yahei 等字体在 Mac 上的网页显示效果较差，应避免。最后，假如所有已声明字体都不适用（这种情况很少），则调用系统默认无衬线字体（sans-serif）。

参考：

[Type is beautiful](http://www.typeisbeautiful.com/2010/03/2190/)

[W3school CSS](http://www.w3schools.com/css/css_font.asp)