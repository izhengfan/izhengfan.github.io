---
layout: cndefault
permalink: /cn/
title: 无求备斋笔记
---

<article>
<blockquote><p>
料知命中无大事，关心雪后有梅花。
</p></blockquote>
</article>

<p style="text-align:left;margin-top:1.2em;margin-bottom:0;">
<b>文章 </b>
| 按<a href="/cnarchive#tags">标签</a>浏览 
<!--<span style="float:right;">按<a href="/cnarchive#tags">标签</a>浏览</span>-->
</p>
---

<table>
{% for post in site.categories.cn %}
<tr id="blog-table">
<td>{{ post.date | date: "%Y-%m-%d" }}</td>
<td><a class="post-list-item" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></td>
</tr>
{% endfor %}
</table>
<hr>
<p>博文<a href="/cnarchive">归档</a></p>
