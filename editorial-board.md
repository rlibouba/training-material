---
layout: base
title: GTN Editorial Board
---

{% assign contributors = site.data['contributors'] %}
{% assign sorted_topics = site | list_topics_ids %}

<h1> GTN Editorial Board </h1>
<section>
 <p class="lead">
  A big <em>Thank You!</em> to the GTN Editorial Board members who ensure the quality of the tutotials in their respective topics.
 </p>

{% for t in sorted_topics %}
 {% assign topic = site.data[t] %}
 <h3> {{ topic.title }}</h3>
 {% include _includes/contributor-list.html contributors=topic.editorial_board badge=true %}

{% endfor %}

</section>
