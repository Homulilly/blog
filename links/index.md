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

<br />
<!-- <br />
<p class="site-title" style="text-align: center">BOOKMARK</p> -->

---- 

<!-- <div class="links-content">
    <div class="link-navigation">
        {% for link in site.data.linkb %}
        <div class="card"><img class="ava nomediumzoom" src="{{ link.avatar }}" />
            <div class="card-header">
                <div><a href="{{ link.site }}" target="_blank"> {{ link.name }}</a> </div>
                <div class="info">{{ link.info }}</div>
            </div>
        </div>
        {% endfor %}
    </div>
</div> -->

<br />

> **本站信息** 
名称： Homulilly
介绍： Aroes's Blog | \萌脇舞以/
网址： https://homulilly.com
头像： https://homulilly.com/images/avatar.jpg
