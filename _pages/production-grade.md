---
title: "Production Grade"
layout: single
permalink: /case-studies/
author_profile: true
---

Case dimana semua hal akan mengikuti best practice,production grade,dan sesuai dengan penerapan konsep masing-masing,contohnya playbook idempotent,terraform clean,dll.
{% for case in site.case_studies %}
<div class="case-preview">
  <h3><a href="{{ case.url | relative_url }}">{{ case.title }}</a></h3>
  <p><em>{{ case.infra_used }}</em> — {{ case.date | date: "%B %Y" }}</p>
</div>
{% endfor %}
