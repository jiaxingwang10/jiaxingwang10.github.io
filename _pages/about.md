---
layout: about
# 💡 诀窍 1：把原本的 title: about 改成你的名字！这样它就会作为大标题显示，并且撑开顶部的安全距离。
title: Jiaxing Wang (王家兴)
permalink: /

# 💡 诀窍 2：使用官方自带的 profile 模块，它会自动帮你把图片放在左边，文字排好。
profile:
  align: left
  image: prof_pic.jpg
  image_circular: false # 如果你想要圆形照片，把这里改成 true 即可
  more_info: >
    <p>Researcher, JD.com</p>
    <p>Beijing, China</p>
    <p><strong>Email:</strong> wjiaxing94 [at] gmail.com</p>

selected_papers: true 
social: true 

announcements:
  enabled: true 
  scrollable: true 
  limit: 5 

latest_posts:
  enabled: false
---

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
    const papersSection = document.querySelector('.publications');
    const servicesSection = document.getElementById('services-section');
    if (papersSection && servicesSection) {
      papersSection.insertAdjacentElement('afterend', servicesSection);
    }
  });
</script>
