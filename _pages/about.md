---
layout: about
title: about
permalink: /

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
  /* 1. 彻底干掉默认的巨大标题 */
  .post-header {
    display: none !important;
  }
  
  /* 2. 完美解决导航栏靠左的问题，直接针对 al-folio 的底层 ID 发号施令 */
  #navbarNav {
    justify-content: flex-end !important;
  }

  /* 3. 社交图标样式 */
  .profile-info .contact-icons {
    font-size: 2rem; 
    text-align: left;
    margin-top: 15px;
  }
  .profile-info .contact-icons a {
    margin-right: 15px; 
  }
</style>

<div class="row mt-5" style="margin-top: 5rem !important; align-items: center;">
  
  <div class="col-sm-4">
    <img src="{{ 'prof_pic.jpg' | prepend: '/assets/img/' | relative_url }}" class="img-fluid z-depth-1 rounded" alt="profile picture" style="width: 100%; max-width: 250px;">
  </div>

  <div class="col-sm-8 profile-info">
    <h1 style="margin-top: 0; font-size: 2.8rem; font-weight: bold;">Jiaxing Wang <span style="font-weight: normal; font-size: 2.2rem;">(王家兴)</span></h1>
    <p style="font-size: 1.2rem; margin-bottom: 5px;">Researcher, JD.com</p>
    <p style="font-size: 1.1rem; margin-bottom: 5px; color: var(--global-text-color-light);">Beijing, China</p>
    <p style="font-size: 1.1rem; margin-bottom: 15px;"><strong>Email:</strong> wjiaxing94 [at] gmail.com</p>

    <div class="social" style="font-size: 2rem; margin-top: 15px;">
      <div class="contact-icons">
        <a href="https://scholar.google.com/citations?user=EHGfstcAAAAJ&hl=en" title="Google Scholar" target="_blank"><i class="ai ai-google-scholar"></i></a>
        <a href="https://github.com/jiaxingwang10" title="GitHub" target="_blank"><i class="fab fa-github"></i></a>
        <a href="https://www.linkedin.com/in/..." title="LinkedIn" target="_blank"><i class="fab fa-linkedin"></i></a>
      </div>
    </div>
  </div>
</div>

I am currently a researcher at JD.com. I obtained my Ph.D. degree from the Chinese Academy of Sciences, Institute of Automation, supervised by [Prof. Jian Cheng](https://clab.ia.ac.cn/en), and the B.Eng. Degree from North China Electric Power University.

My research focuses on Data-centric AI and AI Infrastructure, with a specific emphasis on data mixture optimization, data selection, and data synthesis to construct a self-evolving data flywheel, as well as efficient model training and inference. These techniques have been successfully deployed in production within JD Retail's commodity understanding, yielding substantial business impact.


<div id="services-section" style="margin-top: 40px;">
  <h2><span style="font-weight: bold;">Services</span></h2>
  <hr>
  <ul style="list-style-type: none; padding-left: 0; line-height: 1.8;">
    <li style="margin-bottom: 10px;">
      <i class="fas fa-check-circle" style="color: var(--global-theme-color); margin-right: 8px;"></i>
      <strong>Program Committee Member / Reviewer:</strong> NeurIPS 21-26, ICML 22-26, ICLR 22-26, ACL 26
    </li>
  </ul>
</div>

<script>
  document.addEventListener("DOMContentLoaded", function() {
    // 找到主题自动生成的代表作区域 (.publications)
    const papersSection = document.querySelector('.publications');
    const servicesSection = document.getElementById('services-section');
    
    // 把 Services 强行“传送”到代表作的下方
    if (papersSection && servicesSection) {
      papersSection.insertAdjacentElement('afterend', servicesSection);
    }
  });
</script>
