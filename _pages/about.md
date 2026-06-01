---
layout: about
title: about
permalink: /

# 注意：这里我们删除了 profile 模块，不再使用默认的图片渲染方式
selected_papers: true 
social: true 

announcements:
  enabled: true 
  scrollable: true 
  limit: 5 

latest_posts:
  enabled: true
  scrollable: true 
  limit: 3 
  
---

<style>
  /* 隐藏默认标题 */
  .post-header {
    display: none !important;
  }
  /* 微调社交图标的样式 */
  .profile-info .contact-icons {
    font-size: 2rem; /* 控制图标大小 */
    text-align: left;
    margin-top: 15px;
  }
  .profile-info .contact-icons a {
    margin-right: 15px; /* 图标之间的间距 */
  }
</style>

<div class="row" style="margin-bottom: 40px; align-items: center;">
  
  <div class="col-sm-4">
    <img src="{{ 'prof_pic.jpg' | prepend: '/assets/img/' | relative_url }}" class="img-fluid z-depth-1 rounded" alt="profile picture" style="width: 100%; max-width: 250px;">
  </div>

  <div class="col-sm-8 profile-info">
    <h1 style="margin-top: 0; font-size: 2.8rem; font-weight: bold;">Jiaxing Wang <span style="font-weight: normal; font-size: 2.2rem;">(王家兴)</span></h1>
    <p style="font-size: 1.2rem; margin-bottom: 5px;">Researcher, JD.com</p>
    <p style="font-size: 1.1rem; margin-bottom: 5px; color: var(--global-text-color-light);">Beijing, China</p>
    <p style="font-size: 1.1rem; margin-bottom: 15px;"><strong>Email:</strong> wjiaxing94 [at] gmail.com</p>

    <div class="contact-icons">
      {% include social.html %}
    </div>
  </div>
</div>



I am currently a researcher at JD.com. I obtained my Ph.D. degree from the Chinese Academy of Sciences, Institute of Automation, supervised by [Prof. Jian Cheng](https://clab.ia.ac.cn/en), and the B.Eng. Degree from North China Electric Power University.

My research focuses on Data-centric AI and AI Infrastructure, with a specific emphasis on data mixture optimization, data selection, and data synthesis to construct a self-evolving data flywheel, as well as efficient model training and inference. These techniques have been successfully deployed in production within JD Retail's commodity understanding, yielding substantial business impact.


<!-- Put your address / P.O. box / other info right below your picture. You can also disable any of these elements by editing `profile` property of the YAML header of your `_pages/about.md`. Edit `_bibliography/papers.bib` and Jekyll will render your [publications page](/al-folio/publications/) automatically. -->

<!-- Link to your social media connections, too. This theme is set up to use [Font Awesome icons](https://fontawesome.com/) and [Academicons](https://jpswalsh.github.io/academicons/), like the ones below. Add your Facebook, Twitter, LinkedIn, Google Scholar, or just disable all of them.-->
