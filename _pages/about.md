---
layout: about
title: About
permalink: /

selected_papers: false
social: false

announcements:
  enabled: false
  scrollable: false
  limit: 5 

latest_posts:
  enabled: false
  scrollable: true 
  limit: 3 
---

<style>
  .post-header {
    display: none;
  }
  #navbar > .container {
    max-width: 930px;
    padding: 0.65rem 1rem;
    justify-content: flex-end;
  }
  #navbar .nav-link {
    font-size: 1.05rem;
  }
  #navbar .navbar-toggler {
    margin-left: auto;
  }
  .profile-hero {
    display: grid;
    grid-template-columns: minmax(180px, 250px) minmax(0, 1fr);
    align-items: center;
    gap: 2.5rem;
    padding: 40px 0;
  }
  .profile-photo img {
    display: block;
    width: 100%;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
  .profile-info {
    min-width: 0;
  }
  .profile-info h1 {
    margin-top: 0;
    font-size: clamp(2rem, 4vw, 2.8rem);
    font-weight: bold;
    line-height: 1.2;
  }
  .profile-name-zh {
    display: inline-block;
    font-size: 0.78em;
    font-weight: normal;
    white-space: nowrap;
  }
  .profile-info .contact-icons {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    font-size: 2rem;
    margin-top: 15px;
  }
  .profile-blog-link {
    margin: 1.25rem 0 1.75rem;
  }
  @media (min-width: 576px) {
    #navbar .navbar-collapse {
      margin-left: auto;
      flex-grow: 0;
    }
  }
  @media (max-width: 767px) {
    .profile-hero {
      grid-template-columns: 180px minmax(0, 1fr);
      gap: 1.5rem;
    }
  }
  @media (max-width: 575px) {
    .profile-hero {
      grid-template-columns: minmax(0, 1fr);
      gap: 1.25rem;
      padding: 24px 0 28px;
    }
    .profile-photo {
      max-width: 150px;
    }
    .profile-name-zh {
      display: block;
      margin-top: 0.3rem;
    }
    .news .table th {
      min-width: 5.5rem;
      font-size: 0.85rem;
    }
  }
</style>

<div class="profile-hero">
  
  <div class="profile-photo">
    <img src="{{ 'prof_pic.jpg' | prepend: '/assets/img/' | relative_url }}" alt="Jiaxing Wang">
  </div>

  <div class="profile-info">
    <h1>Jiaxing Wang <span class="profile-name-zh">(王家兴)</span></h1>
    <p style="font-size: 1.2rem; margin-bottom: 5px;">Researcher, JD.com</p>
    <p style="font-size: 1.1rem; margin-bottom: 5px; color: var(--global-text-color-light);">Beijing, China</p>
    <p style="font-size: 1.1rem; margin-bottom: 15px;"><strong>Email:</strong> wjiaxing94 [at] gmail.com</p>

    <div class="social" style="font-size: 2rem; margin-top: 15px;">
      <div class="contact-icons">
        <a href="https://scholar.google.com/citations?user=EHGfstcAAAAJ&hl=en" title="Google Scholar" target="_blank"><i class="ai ai-google-scholar"></i></a>
        <a href="https://github.com/jiaxingwang10" title="GitHub" target="_blank"><i class="fab fa-github"></i></a>
        <a href="https://www.linkedin.com/in/jiaxing-wang-bb289b3a5/" title="LinkedIn" target="_blank"><i class="fab fa-linkedin"></i></a>
      </div>
    </div>
  </div>
</div>
I am currently a researcher at JD.com. I obtained my Ph.D. degree from [National Laboratory of Pattern Recognition](https://nlpr.ia.ac.cn/) from [CASIA](https://ia.cas.cn), supervised by [Prof. Jian Cheng](https://clab.ia.ac.cn/en), and the B.Eng. Degree from North China Electric Power University.

My research focuses on how training data shapes what models learn, and how to make learning more efficient. I study data mixture optimization, selection, and synthesis, with an emphasis on connecting target-driven objectives to practical training algorithms. This work provides a foundation for self-evolving AI systems, where feedback from model failures guides iterative improvements to training data and model capabilities.

At JD Retail, I have deployed a data flywheel for product understanding that connects target evaluation, data generation and selection, and model training. My work also includes systems for efficient training and inference.

<p class="profile-blog-link">For an overview of my recent work, see <a href="{{ '/blog/#designing-series-title' | relative_url }}">Designing Training Data for Language Models</a>, a three-part essay series on training data and learning.</p>

<h2>
  <a href="{{ '/news/' | relative_url }}" style="color: inherit">News</a>
</h2>
{% include news.liquid limit=true %}

<h2>
  <a href="{{ '/publications/' | relative_url }}" style="color: inherit">Selected Publications</a>
</h2>
{% include selected_papers.liquid %}

<div id="services-section" style="margin-top: 40px;">
  <h2><span style="font-weight: bold;">Services</span></h2>
  <hr>
  <ul style="list-style-type: none; padding-left: 0; line-height: 1.8;">
    <li style="margin-bottom: 10px;">
      <i class="fas fa-check-circle" style="color: var(--global-theme-color); margin-right: 8px;"></i>
      <strong>Program Committee Member / Reviewer:</strong> NeurIPS 21-26, ICML 22-26, ICLR 22-26, ACL 26
    </li>
    <li style="margin-bottom: 10px;">
      <i class="fas fa-award" style="color: var(--global-theme-color); margin-right: 8px;"></i>
      <strong>Top Reviewer:</strong> NeurIPS 2026
    </li>
  </ul>
</div>
