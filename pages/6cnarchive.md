---
layout: cndefault
title: 归档 (含英文博客)
permalink: /cnarchive/
---

### 归档 (含英文博客)

---

标签: {% for tag in site.tags %}<block class="tag"><a href="#{{ tag | first }}">{{ tag | first }} </a></block>{% endfor %}
{% for post in site.posts  %}{% capture this_year %}{{ post.date | date: "%Y" }}{% endcapture %}
{% capture this_month %}{{ post.date | date: "%m" }}{% endcapture %}
{% capture next_year %}{{ post.previous.date | date: "%Y" }}{% endcapture %}
{% capture next_month %}{{ post.previous.date | date: "%m" }}{% endcapture %}
{% if forloop.first %}<legend id="{{this_year}}">{{this_year}}</legend><ul>{% endif %}
<p><span>{{ post.date | date: "%Y-%m-%d" }}</span> <a class="pjaxlink" href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></p>
{% if forloop.last %}</ul>{% else %}{% if this_year != next_year %}</ul><legend id="{{next_year}}">{{next_year}}</legend><ul>{% endif %}{% endif %}
{% endfor %} 
<h3 id="tags">标签</h3>
<p>{% for tag in site.tags %}<block class="tag"><a href="#{{ tag | first }}">{{ tag | first }} </a></block>{% endfor %}</p>
{% for tag in site.tags %}
  <div>
	<legend id="{{ tag | first }}">{{ tag | first }}</legend>
	<ul>{% for posts in tag  %}{% for post in posts %}{% if post.url %}
  <p><span>{{ post.date | date: "%Y-%m-%d" }}</span> <a class="pjaxlink" href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></p>
  {% endif %}{% endfor %}{% endfor %}</ul>
  </div>
{% endfor %}
