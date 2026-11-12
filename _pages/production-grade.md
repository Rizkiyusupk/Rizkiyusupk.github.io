---
title: "Production Grade"
layout: single
permalink: /production-grade/
author_profile: true
---

page dimana semua hal akan mengikuti best practice,production grade,dan sesuai dengan penerapan konsep masing-masing,contohnya playbook idempotent,terraform clean,dll.
{% for case in site.production_grade %}
<div class="procuction-preview">
  <h3><a href="{{ production.url | relative_url }}">{{ production.title }}</a></h3>
  <p><em>{{ production.infra_used }}</em> — {{ production.date | date: "%B %Y" }}</p>
</div>
{% endfor %}
