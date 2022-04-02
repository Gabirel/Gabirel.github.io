---
layout: default
title: Categories
---

<div id="archives">
{% assign categories = site.categories | sort %}
{% for category in categories %}
  <div class="archive-group">
    {% capture category_name %}{{ category | first }}{% endcapture %}
    <div id="#{{ category_name | slugize }}"></div>
    <p></p>
    <h4 class="category-head">{{ category_name }}</h4>
    {% for post in site.categories[category_name] %}
    <article class="archive-item">
      <div class="archive-item-date">
        <span >{{ post.date | date: "%Y-%m-%d" }}</span>
      </div>
      <div class="archive-item-post">
        <a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a>
      </div>
    </article>
    {% endfor %}
  </div>
{% endfor %}
</div>
