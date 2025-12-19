---
layout: post
title:  Font Family Introduction
date: 2015-08-14 22:00:51 
categories: en
tags: type web
---

__Contents__

* content
{:toc}

Type is a rather general and informative concept. In this post we are mainly discussing how to choose proper type style for web display, so we borrow the phrase 'font family' from CSS, just to make our discussion more specific.

### Serif and Sans-serif

![lXHpTg.jpg](https://s2.ax1x.com/2020/01/15/lXHpTg.jpg)

One of the most important concept in font is 'serif', which means a short line at the end of the main strokes of a character. Usually it acts as a decotation part in the character to make the whole body look more elegant and graceful, and increase readability. Therefore, the noun 'Serif' also refers to a font group featured with serif. In contrast, 'Sans-serif' represents a font group without serif decoration. In CSS, such font group is named 'generic family', and a specific font family with unique design style is called 'font family'. Apart from Serif and Sans-serif, Monospace is also a generic family, in which every character covers the same width. 

<!-- ![](/images/font-family.png) -->
![lXHifs.png](https://s2.ax1x.com/2020/01/15/lXHifs.png)

Different generic families have different appropriate applications. Generally speaking, Serif looks good in printed situations, with varied serif designs resulting in stylish visual effects, thus bringing more functional and subtle feeling to the readers. In digital screen, however, since all shapes are presented with combined discrete dots, the display effect of a character is limited by screen resolution. Serif fonts, especially in low-resolution screens, tends to be too sawtooth-shaped or too slender to be easily recognised by the readers, especially for Chinese characters. Therefore, Sans-serif fonts, like Arial and Microsoft Yahei, with strong and clear main strokes, is a better choice in digital screen display, which is also why it is widely used in web design. Monospace fonts are often adopted in programming editors and webpage code blocks. 

### Legibility and Readability

For a font family, legibility defines how easy it is to recognize what a character is. The most typical sample is the design of character captital 'I', lowercase 'l' and number '1', as shown in the picture below.

<!-- ![](/images/font-legi.png) -->
![lXHPYj.png](https://s2.ax1x.com/2020/01/15/lXHPYj.png)

As you can see, for some font families like Consolas, you can easily tell any one of these 3 characters from each other, which is not the case for other ones like Arial (captital 'I' & lowercase 'l') and Times New Roman (lowercase 'l' & number '1'). Legibility should be closely considered in those situations where indivisual character accuracy is significant, like showing password, and programming. That is why many programmers choose Consolas as the default font of their editors.

Readability is a more general concept, which is not only affected by the font design style, but also many other aspects in typography. A webpage with properly chosen font family may also looks ugly with poor-skilled typesetting.

### Cross Platform Consideration

General principle for webpage font selection: Sans-serif rather than Serif. However, different OS have different different font rendering strategy. For example, Mac's principle is to display the original shape of font design, which makes font display in high-resolution screen very graceful while in low-resolution screen usually too slender to read. On the other hand, Windows values universal legibility more than elegance and tends to use high hinting to 'reconstruct' the font design so that all characters can be easily recognised even in very low resolution screens. The side effect of this strategy is that Windows font display in high-resolution screen is comparatively poor. Besides, different built-in font familiese in different OS also leads to display difference.  

Popular OS includes Windows, Mac OS X and Linux, which are what we consider here. In this website, the selected font families are shown in the CSS code below:

```css
body {
    font-family: "Georgia","Hiragino Sans GB", "Droid Sans Fallback", "Microsoft JhengHei", "Microsoft YaHei",sans-serif;
}
```

CSS will use the font family declared at the beginning of the `font-family` list. If the first one does not exist in current OS or not applicable, CSS will use the next one. Since usually a Latin font family does not contain Chinese character set, while Chinese font families generally contain Latin, it is advisable to first declared Latin font family followed by Chinese or other language font sets. In our case, we select Georgia for English display, which is a common well designed font with comparatively good legibility and graceful structure. For Chinese fonts, things are more complicated, due to different built-in Chinese font sets in different OS.  Hiragino Sans GB is for Mac and Droid Sans Fallback is for Linux, both of which are well designed and widely accepted Sans-serif fonts. Yahei and Jhenghei are common in Windows machines. The reason to put Mac and Linux fonts before Windows ones is that some Mac may contain Windows fonts because of installation of Microsoft Office Software, but these fonts perform poorly in web display in Mac and should be prevented. Finally, in the case of all the prefered font families not existing, which rarely happens, the default Sans-serif font of current OS will be adopted. 
 
 References:
  
[Type is beautiful](http://www.typeisbeautiful.com/2010/03/2190/)

[W3school CSS](http://www.w3schools.com/css/css_font.asp)
  
