---
title: "Algorithm"
layout: archive
permalink: categories/Algorithm
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.Algorithm %}

<div class="entries-list">
	{% for post in posts %}
		{% include archive-single2.html %}
	{% endfor %}
</div>
