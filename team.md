---
layout: page
title: Team
permalink: /team/
---

# 认识 A_Bit_S 团队

我们围绕实际项目积累经验，在不同领域彼此支撑。

---

## Core Members

{% for author_entry in site.data.authors %}
  {% assign author_key = author_entry[0] %}
  {% assign author = author_entry[1] %}
  {% unless author_key == 'bg' %}
### {{ author.name }} {{ author.display_name }} - {{ author.role }}
**领域**：{{ author.areas | join: '、' }}

{{ author.bio }}

**近期关注**：{{ author.focus }}

---

  {% endunless %}
{% endfor %}

## 非 IT 合作伙伴

{% assign bg = site.data.authors.bg %}
### {{ bg.name }} {{ bg.display_name }} - {{ bg.role }}

{{ bg.bio }}

**近期关注**：{{ bg.focus }}

---

## 协作方式

我们围绕同一个目标协作，频繁结对、互审代码、持续学习，共同推动项目前进。
