---
title: "Publication"
layout: archive
permalink: categories/publication
author_profile: false
sidebar: 
    nav: "nav"
---

<hr>
{% assign posts = site.categories.publication %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}