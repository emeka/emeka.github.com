---
layout: default
tagline: Supporting tagline
---
{% include JB/setup %}


<div class="blog-index">
  {% for post in site.posts %}
  {% assign content = post.content %}
  {% include post_detail.md %}
  {% endfor %}
</div>

