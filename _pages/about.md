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
  /* 1. 彻底隐藏默认标题 */
  .post-header {
    display: none !important;
  }
  
  /* 2. 导航栏：顶满页面顶部，链接靠右，与内容区域右边对齐 */
  header, nav, #navbar, nav.navbar, .navbar {
    top: 0 !important;
    left: 0 !important;
    right: 0 !important;
    margin: 0 !important;
    padding-top: 0 !important;
    padding-bottom: 0 !important;
    width: 100% !important;
  }
  nav.navbar, .navbar {
    display: flex !important;
    justify-content: center !important;
  }
  nav .container, nav .container-fluid,
  .navbar .container, .navbar .container-fluid,
  .navbar > .container {
    display: flex !important;
    justify-content: flex-end !important;
    max-width: 930px !important;
    width: 100% !important;
    margin: 0 auto !important;
    padding: 8px 0 !important;
  }
  header nav ul, nav.navbar ul, .navbar-nav,
  .navbar-collapse, #navbarNav {
    margin-left: auto !important;
    margin-right: 0 !important;
    display: flex !important;
    justify-content: flex-end !important;
    flex-wrap: wrap !important;
  }
  .navbar-brand, nav .brand {
    display: none !important;
  }
  /* 导航栏字体稍大 */
  nav a, .navbar a, .nav-item a, .nav-link {
    font-size: 1.05rem !important;
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

<div style="padding-top: 40px; padding-bottom: 40px; display: flex; flex-wrap: wrap; align-items: center; justify-content: space-between;">
  
  <div style="width: 32%; min-width: 200px; padding-right: 20px;">
    <img src="{{ 'prof_pic.jpg' | prepend: '/assets/img/' | relative_url }}" alt="profile picture" style="width: 100%; max-width: 250px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">
  </div>

  <div class="profile-info" style="width: 65%; flex-grow: 1; min-width: 300px;">
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
I am currently a researcher at JD.com. I obtained my Ph.D. degree from [National Laboratory of Pattern Recognition](https://nlpr.ia.ac.cn/) from [CASIA](https://ia.cas.cn), supervised by [Prof. Jian Cheng](https://clab.ia.ac.cn/en), and the B.Eng. Degree from North China Electric Power University.

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
    const papersSection = document.querySelector('.publications');
    const servicesSection = document.getElementById('services-section');
    if (papersSection && servicesSection) {
      papersSection.insertAdjacentElement('afterend', servicesSection);
    }
  });
</script>
