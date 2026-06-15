---
layout: about
title: about
permalink: /
subtitle: B.S. in Electrical and Computer Engineering, <a href='https://en.snu.ac.kr/'>Seoul National University</a>.

profile:
  align: right
  image: prof_pic.jpg
  image_circular: false # crops the image to make it circular
  more_info:

selected_papers: false # flip to true once you add a paper marked selected={true} — it then shows here on the homepage
social: true # includes social icons at the bottom of the page

announcements:
  enabled: true # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: false # turn on once you start writing blog posts
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---

I'm a fourth-year undergraduate graduating in February 2027. I work on improving the performance of machine learning models, coming from a background in algorithmic acceleration — where the whole challenge is to speed models up while preserving their quality. That fight to hold on to quality is what pulled me toward improving it directly, and an efficiency-minded background still shapes how I approach it.

I'm currently a research intern at the VLSI Lab (Prof. Jae-Joon Kim), and this summer I'll be joining DSAIL (Prof. Sungroh Yoon) to work on multimodal learning.

<!-- Clear the floated profile photo so the sections below start full-width beneath it. -->
<div style="clear: both;"></div>

## publications

<style>
  /* Highlight my own name in the publications list as bold (theme default underlines it). */
  .author em { font-style: normal; font-weight: 700; border-bottom: 0 !important; }
</style>

<div class="publications">
  {% bibliography %}
</div>

## projects

<div class="projects">
  <div class="row row-cols-1 row-cols-md-2">
    {% assign sorted_projects = site.projects | sort: "importance" %}
    {% for project in sorted_projects %}
      {% include projects_horizontal.liquid %}
    {% endfor %}
  </div>
</div>
