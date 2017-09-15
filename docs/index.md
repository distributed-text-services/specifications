---
layout: page
title: Distributed Text Services
---
{% include JB/setup %}

## About the project

Definition of DTS goals and history ?

## Contact Info

Google Group: https://groups.google.com/forum/#!forum/distributed-text-services

## About the members

Here is a list of our contributors

Bridget Almas, Tufts University

Frederik Baumgardt, Tufts University

Hugh Cayless, Duke University

Thibault Clérice, Universität Leipzig

Andrew Dunning, University of Toronto

Pietro Liuzzo, Universität Hamburg

Paolo Monella, Università degli Studi di Palermo

Emmanuelle Morlock, L'ENS de Lyon

Jonathan Robie, BiblicalHumanities.org

Jeffrey Witt, Loyola University

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## Forum

<iframe id="forum_embed"
  src="javascript:void(0)"
  scrolling="no"
  frameborder="0"
  width="900"
  height="700">
</iframe>
<script type="text/javascript">
  document.getElementById('forum_embed').src =
     'https://groups.google.com/forum/embed/?place=forum/distributed-text-services'
     + '&showsearch=true&showpopout=true&showtabs=false'
     + '&parenturl=' + encodeURIComponent(window.location.href);
</script> 
