{% include JB/setup %}

<div class="content row">
    <div class="page-header">
        <h1><a href="{{ root_url }}{{ post.url }}">{{ post.title }}</a></h1>
        <div class="date"><span>{{ post.date | date_to_long_string }}</span></div>
    </div>
    {{ content }}
</div>
