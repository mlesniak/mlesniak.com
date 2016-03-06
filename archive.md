---
layout: page
title: Archive
permalink: /archive/
---

  <h4>2016</h4>
  <ul class="archive-post">
    {% for post in site.posts %}
      <li>
      	<a class="" href="{{ post.url | prepend: site.baseurl }}">	
      		{{ post.title }}
      	</a>
      </li>
    {% endfor %}
  </ul>
