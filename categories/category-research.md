---
title: "Research"
layout: archive
permalink: categories/research
author_profile: false
sidebar: 
    nav: "nav"
---

<hr>
{% assign posts = site.categories.research %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}