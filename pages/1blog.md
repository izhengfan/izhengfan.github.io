---
layout: default
permalink: /blog/
---

<div class="row">
  
  <div class="col-sm-1">
    <div id="left-bar">
      <p><a href="{{ site.baseurl }}/">Home</a></p>
        {% if page.url == "/blog/" %}
          {% else %}
        {% endif %}
          <!--<a href="{{ site.baseurl }}/"><span class="glyphicon glyphicon-th-large"></span>Blog</a></li>-->
          <p><a href="{{ site.baseurl }}/blog/">Blog</a></p>

        {% for p in site.pages %}
          {% if p.title %}
            {% if p.url == page.url %}
              {% else %}
            {% endif %}
            <p><a href="{{ p.url | prepend: site.baseurl }}"> {{ p.title }}</a></p>
          {% endif %}
        {% endfor %}
    </div>
  </div>
  
  
  <div class="col-sm-8 col-sm-offset-1">
    <div class="post-area">
      <div class="post-list-body">
        <div class="all-posts" post-cate="All">
          <table>
          {% for post in site.posts %}
            <tr id="blog-table">
            <td>{{ post.date | date: "%Y-%m-%d" }}</td>
            <td><a class="post-list-item" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></td>
            </tr>
          {% endfor %}
          </table>
        </div>
        {% for category in site.categories %}
          <div post-cate="{{category | first}}">{% for posts in category  %}
            <table>
              {% for post in posts %}
              {% if post.url %}
              <tr id="blog-table">
                <td>{{ post.date | date: "%Y-%m-%d" }}</td>
                <td> <a href="{{ post.url | prepend: site.baseurl }}" class="post-list-item">{{ post.title }}</a></td>
              </tr>
              {% endif %}
              {% endfor %}
            </table>
            {% endfor %}
          </div>
        {% endfor %}
      </div>
    </div>
  </div>
  
  
  <div class="col-sm-2">
    <div class="shadow-corner-curl hidden-xs">
      <div class="categories-list-header">
        Tags
      </div>
      
      <a href="javascript:;" class="categories-list-item" cate="All">
        All<span class="my-badge"> {{site.categories | size}}</span>
      </a>
      {% for category in site.categories %}
        <a href="javascript:;" class="categories-list-item" cate="{{ category | first }}">
          {{ category | first }} <span class="my-badge">{{ category | last | size }}</span>
        </a>
      {% endfor %}
    </div>
  </div>
</div>