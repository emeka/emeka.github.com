{% include JB/setup %}
<h1 class="entry-title">
{% if page.title %}
    <a href="{{ root_url }}{{ page.url }}">{{ page.title }}</a>
{% endif %}
{% if post.title %}
    <a href="{{ root_url }}{{ post.url }}">{{ post.title }}</a>
{% endif %}
</h1>
<div class="date"><span>{{ post.date | date_to_long_string }}</span></div>

<div class="entry-content">{{ content }}</div>