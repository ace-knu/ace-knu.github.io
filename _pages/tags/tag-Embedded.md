---
title: "Embedded"
layout: archive
permalink: tags/Embedded
author_profile: false
sidebar: 
    nav: "nav"
---

{% assign posts = site.tags.Embedded %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}