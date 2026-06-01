---
layout: about
title: about
permalink: /
# 1. 这里填你的副标题，记得去掉那对多余的引号
subtitle: Researcher, JD.com · Beijing, China · wjiaxing94@gmail.com

profile:
  align: left           # 👈 改回左边，实现顶格
  image: prof_pic.jpg
  image_circular: false
  # address 已经包含在 subtitle 里了，这里留空，防止照片下面出现冗余文字
  more_info: >

# 2. 以下完全保留你刚才贴的原始控制逻辑，保证 News 和 Papers 正常显示
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

# 3. 关键：注入这段 CSS，强制把“姓名栏”拉到照片右边
style: |
  .profile {
    float: left;
    width: 200px !important;
    margin-top: 0 !important;
    margin-right: 30px !important;
  }
  .post-header {
    display: block;
    overflow: hidden;
    margin-top: 0 !important;
    padding-top: 0 !important;
  }
  header h1 {
    margin-top: 0 !important;
    line-height: 1.2;
  }
---

I am currently a researcher at JD.com. I obtained my Ph.D. degree from the Chinese Academy of Sciences, Institute of Automation, supervised by [Prof. Jian Cheng](https://clab.ia.ac.cn/en), and the B.Eng. Degree from North China Electric Power University.

My research focuses on Data-centric AI and AI Infrastructure, with a specific emphasis on data mixture optimization, data selection, and data synthesis to construct a self-evolving data flywheel, as well as efficient model training and inference. These techniques have been successfully deployed in production within JD Retail's commodity understanding, yielding substantial business impact.


<!-- Put your address / P.O. box / other info right below your picture. You can also disable any of these elements by editing `profile` property of the YAML header of your `_pages/about.md`. Edit `_bibliography/papers.bib` and Jekyll will render your [publications page](/al-folio/publications/) automatically. -->

<!-- Link to your social media connections, too. This theme is set up to use [Font Awesome icons](https://fontawesome.com/) and [Academicons](https://jpswalsh.github.io/academicons/), like the ones below. Add your Facebook, Twitter, LinkedIn, Google Scholar, or just disable all of them.-->
