---
layout: home
tagline: Supporting tagline
---
{% include JB/setup %}


{% for post in site.posts %}
{% assign content = post.content %}
{% include post_summary.md %}
{% endfor %}


