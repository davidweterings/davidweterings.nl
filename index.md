---
layout: page
title: Welcome to my website!
tagline: Supporting tagline
---
{% include JB/setup %}

Still learning to use Jekyll, site should be good soon &trade;.

## About me

Currently finishing my masters in Game and Media Technology at the University of Utrecht. Interested in C++, game development, optimization and low-level programming.

You can read my resume [here](resume.html).

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

