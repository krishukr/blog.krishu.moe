---
title: 友链
date: 2022-05-16 18:59:28
toc:
  enable: false
---

<div id="links">
<div class="links-content">
<div class="link-navigation">
{% for link in site.data.links %}
<div class="card">
  <a href="{{ link.site }}" target="_blank">
  <img class="ava" src="{{ link.avatar }}"/></a>
  <div class="card-header">
  <div><a href="{{ link.site }}" target="_blank">{{ link.name }}</a></div>
  <div class="info" title="{{ link.info }}">{{ link.info }}</div>
  </div>
</div>
{% endfor %}
</div>
</div>
</div>

{% note success %}

## 添加友链

发[邮件](mailto:i@krishu.moe)给我

{% endnote %}
