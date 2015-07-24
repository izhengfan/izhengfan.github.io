---
layout: page
title: Archive
permalink: /archive/
---

{% for post in site.posts  %}
{% capture this_year %}{{ post.date | date: "%Y" }}{% endcapture %}
{% capture this_month %}{{ post.date | date: "%m" }}{% endcapture %}
{% capture next_year %}{{ post.previous.date | date: "%Y" }}{% endcapture %}
{% capture next_month %}{{ post.previous.date | date: "%m" }}{% endcapture %}

{% if forloop.first %}
<legend id="{{this_year}}-{{this_month}}">{{this_year}}-{{this_month}}</legend>
<ul>
{% endif %}
<span>{{ post.date | date: "%Y-%m-%d" }}</span>  
<a class="pjaxlink" href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a><br>

{% if forloop.last %}
</ul>
{% else %}
{% if this_year != next_year %}
</ul>
<legend id="{{next_year}}-{{next_month}}">{{next_year}}-{{next_month}}</legend>
<ul>
{% else %}    
{% if this_month != next_month %}
</ul>
<legend id="{{next_year}}-{{next_month}}">{{next_year}}-{{next_month}}</legend>
<ul>
{% endif %}
{% endif %}
{% endif %}
{% endfor %} 