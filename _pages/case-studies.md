---
title: "Case Studies"
layout: single
permalink: /case-studies/
author_profile: true
---

Eksperimen dan skenario yang dijalankan di atas infrastruktur yang sudah dibangun — mengubah topologi, menambah beban, dan menganalisa bagaimana sistem merespons.

{% for case in site.case_studies %}
<div class="case-preview">
  <h3><a href="{{ case.url | relative_url }}">{{ case.title }}</a></h3>
  <p><em>{{ case.infra_used }}</em> — {{ case.date | date: "%B %Y" }}</p>
</div>
{% endfor %}
