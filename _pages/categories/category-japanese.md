---
title: "日本語"
layout: archive
permalink: categories/japanese
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories["日本語"]%}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}
