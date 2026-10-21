---
layout: page
title: About
permalink: /about/
---

I'm Uday Kiran Pedda, a blockchain and MERN developer focused on secure Web3 protocol design and smart contract auditing.

This site is my research notebook:

- [Research Log](/research-log/): short, dated working notes.
- [Write-ups](/writeups/): longer technical articles.

## Links

{% for link in site.profile_links %}
- [{{ link.name }}]({{ link.url }})
{%- endfor %}
