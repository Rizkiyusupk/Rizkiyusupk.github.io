---
title: "Projects"
layout: single
permalink: /projects/
author_profile: true
---

{% for project in site.data.projects %}
<div class="project-case">
  <h2>{{ project.title }}</h2>
  <p>{{ project.description }}</p>

  <div class="project-phases">
    {% for phase in project.phases %}
    <div class="phase-card">
      <h4><a href="{{ phase.url | relative_url }}">{{ phase.name }}</a></h4>
      <p>{{ phase.description }}</p>
      <small>{{ phase.date | date: "%B %Y" }}</small>
    </div>
    {% endfor %}
  </div>
</div>
<hr>
{% endfor %}
