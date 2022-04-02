---
layout: default
title: すべてのポスト
---

<div class="home" id="home">
  <h1 class="pageTitle">すべてのポスト</h1>
  <div class="posts noList">
    {%- for post in site.posts -%}
    {% if post.lang == "jp" %}
    <article>
        <span class="date">{{ post.date | date: '%B %d, %Y' }}</span>
        <!-- add update time -->
        {% if post.last_update %}
        <span class="date">| 修正: {{ post.last_update | date: '%B %d, %Y' }}</span>
        {% endif %}
        <h3><a class="post-link" href="{{ post.url }}">{{ post.title }}</a></h3>
        <p>{%- if post.description -%}{{ post.description }}{%- else -%}{{ post.excerpt | strip_html }}{%- endif -%}</p>
    </article>
    {% endif %}
    {%- endfor -%}
  </div>
</div>
