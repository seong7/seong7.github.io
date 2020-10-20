---
layout: page
page_title: "Eco 의 개발 공부 로그"
title: Posts
permalink: /posts/
main_nav: true
---

{% for category in site.categories %}
{% capture cat %}{{ category | first }}{% endcapture %}

  <h2 id="{{cat}}">{{ cat | capitalize }}</h2>
  <ul class="posts-list">
  {% for post in site.categories[cat] %}
    <li>
      <strong>
        <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
      </strong>
      <span class="post-date">- {{ post.date | date_to_long_string }}</span>
    </li>
  {% endfor %}
  </ul>
  {% if forloop.last == false %}<hr>{% endif %}
{% endfor %}
<br>
