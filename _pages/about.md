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
  enabled: false # flip to true once you have news to post (papers, talks, projects)
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: false # turn on once you start writing blog posts
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---

I'm a fourth-year undergraduate graduating in February 2027. I work on improving the performance of machine learning models, coming from a background in algorithmic acceleration — where the whole challenge is to speed models up while preserving their quality. That fight to hold on to quality is what pulled me toward improving it directly, and an efficiency-minded background still shapes how I approach it.

I'm currently a research intern at the <a href='https://vlsi.snu.ac.kr/'>VLSI Lab</a> (Prof. <a href='https://scholar.google.com/citations?user=Ee994T0AAAAJ&hl=en'>Jae-Joon Kim</a>), and this summer I'll be joining <a href='https://data.snu.ac.kr/'>DSAIL</a> (Prof. <a href='https://scholar.google.com/citations?user=Bphl_fIAAAAJ&hl=en'>Sungroh Yoon</a>) to work on multimodal learning.

<!-- Clear the floated profile photo so the sections below start full-width beneath it. -->
<div style="clear: both;"></div>

## Publications

<style>
  /* Highlight my own name in the publications list as bold (theme default underlines it). */
  .author em { font-style: normal; font-weight: 700; border-bottom: 0 !important; }
</style>

<div class="publications">
  {% bibliography %}
</div>

## Projects

<style>
  /* Project cards: image fixed at 1/4 of the card width, centered on white so wide
     figures get letterbox padding instead of being cropped or stretched. */
  .project-card {
    display: flex;
    align-items: stretch;
    min-height: 120px;
    margin-bottom: 1.25rem;
    border: 1px solid var(--global-divider-color);
    border-radius: 8px;
    overflow: hidden;
    color: var(--global-text-color);
    text-decoration: none;
    transition:
      box-shadow 0.2s ease,
      transform 0.2s ease;
  }
  .project-card:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 14px rgba(0, 0, 0, 0.12);
  }
  .project-thumb {
    flex: 0 0 25%;
    max-width: 25%;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 0.5rem;
    background: #fff;
  }
  .project-thumb img {
    max-width: 100%;
    max-height: 110px;
    object-fit: contain;
  }
  .project-info {
    flex: 1 1 75%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 1rem 1.25rem;
  }
  .project-info h3 {
    margin: 0 0 0.35rem;
    font-size: 1.2rem;
  }
  .project-card:hover .project-info h3 {
    color: var(--global-theme-color);
  }
  .project-info p {
    margin: 0;
    opacity: 0.85;
  }
  @media (max-width: 576px) {
    .project-card {
      flex-direction: column;
    }
    .project-thumb {
      max-width: 100%;
      flex-basis: auto;
    }
  }
</style>

<div class="projects">
  {% assign sorted_projects = site.projects | sort: "importance" %}
  {% for project in sorted_projects %}
    <a class="project-card" href="{{ project.url | relative_url }}">
      <div class="project-thumb">
        <img src="{{ project.img | relative_url }}" alt="{{ project.title }}" />
      </div>
      <div class="project-info">
        <h3>{{ project.title }}</h3>
        <p>{{ project.description }}</p>
      </div>
    </a>
  {% endfor %}
</div>
