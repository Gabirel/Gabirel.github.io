---
layout: default
title: すべてのポスト
---

<div id="articles">
  <h1>すべてのポスト</h1>
  <ul class="posts noList">
    {% for post in site.posts %}
      {% if post.lang == "jp" %}
      <li>
      	<span class="date">{{ post.date | date_to_string }}</span>
        <!-- add update time -->
        {% if post.last_update %}
        <span class="date"> | 修正: {{ post.last_update | date_to_string }}</span>
        {% endif %}

      	<h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      	<p class="description">{%- if post.description -%}{{ post.description  | strip_html | strip_newlines | truncate: 120 }}{%- else -%}{{ post.content | strip_html | strip_newlines | truncate: 120 }}{%- endif -%}</p>
      </li>
      {% endif %}
    {% endfor %}
  </ul>
</div>
