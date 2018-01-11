---
layout: page
title: 标签列表 
---
<div class="page-content wc-container">
	<div class="post">
		<h1>标签列表</h1>  
		<ul class="tags">
			{% for tag in site.tags %}
			<li><a href="{{site.baseurl}}/tag/{{ tag[0] }}">{{ tag[0] }}</a></li>
			{% endfor %}
		</ul>
	</div>
</div>