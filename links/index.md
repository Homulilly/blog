---
title: 友情链接
type: links
---

---- 
<div class="links-content">
    <div class="link-navigation">
        {% for link in site.data.links %}
        <div class="card"><img class="ava nomediumzoom" src="{{ link.avatar }}" />
            <div class="card-header">
                <div><a href="{{ link.site }}" target="_blank"> {{ link.name }}</a> </div>
                <div class="info">{{ link.info }}</div>
            </div>
        </div>
        {% endfor %}
    </div>
</div>

<h2 class="post-title" style="text-align: center">一些链接</h2>

----

<div class="links-content">
    <div class="link-navigation">
        {% for link in site.data.linksex %}
        <div class="card"><img class="ava nomediumzoom" src="{{ link.avatar }}" />
            <div class="card-header">
                <div><a href="{{ link.site }}" target="_blank"> {{ link.name }}</a> </div>
                <div class="info">{{ link.info }}</div>
            </div>
        </div>
        {% endfor %}
    </div>
</div>

<h2 class="post-title" style="text-align: center">友链格式</h2>

----

```md
- name: Homulilly
  info: Aroes's Blog | \萌脇舞以/
  site: https://homulilly.com/
  avatar: https://homulilly.com/images/avatar.jpg
```

