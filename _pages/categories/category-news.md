---
title: "News"
layout: archive
permalink: categories/news
author_profile: false
sidebar: 
    nav: "nav"
---

<hr>
{% assign posts = site.categories.news %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}

