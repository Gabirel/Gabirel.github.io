---
layout: default
title: Contact Long Haul
---

<div id="contact">
  <h1 class="pageTitle">Contact Me</h1>
  <div class="contactContent">
    <p class="intro">Thank your for trying to contact me. You are welcome to submit your messages.</p>
    <p>If you have questions about the theme feel free to <a href="mailto:{%- if site.data.config.social.email -%} {{ site.data.config.social.email }} {%- else -%} {{ site.social.email }} {%- endif -%}">email me</a> or create an issue on <a href="https://github.com/{%- if site.data.config.social.github -%} {{ site.data.config.social.github | append: ".github.io"}} {%- else -%}  brianmaierjr/long-haul {%- endif -%}">GitHub</a>.</p>
    <p>The form is provided by <a href="http://formspree.io/">Formspree.</a> Follow the directions on their site to set up the form for use.</p>
  </div>
  <form action="http://formspree.io/your@mail.com" method="POST">
    <label for="name">Name</label>
    <input type="text" id="name" name="name" class="full-width"><br>
    <label for="email">Email Address</label>
    <input type="email" id="email" name="_replyto" class="full-width"><br>
    <label for="message">Message</label>
    <textarea name="message" id="message" cols="30" rows="10" class="full-width"></textarea><br>
    <input type="submit" value="Send" class="button">
  </form>
</div>
