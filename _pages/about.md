---
layout: about
title: about
permalink: /
# 注意：subtitle 保持原样，无需引号，干净整洁
subtitle: Researcher, JD.com · Beijing, China · wjiaxing94@gmail.com

profile:
  align: left         # 👈 图片靠左，顶格
  image: prof_pic.jpg
  image_circular: false
  address:            # 留空，删除原来的地址信息

# Jekyll 网页功能的控制中心
# 这些代码原本错误地出现在正文里，现在它们回家了
news: true           # 启用新闻
selected_papers: true # 启用精选论文
social: true          # 启用社交图标

# 自定义样式：实现左图右文对齐，强制顶格
style: |
  .profile {
    float: left;
    width: 200px !important;   /* 调整照片宽度 */
    margin-top: 0 !important;   /* 强制照片顶格 */
    margin-right: 30px !important;
  }
  .post-header {
    display: block;
    overflow: hidden;         /* 文字绕流，不钻到底部 */
    margin-top: 0 !important;
    padding-top: 0 !important;
  }
  header h1 {
    margin-top: 0 !important;  /* 名字顶格 */
  }
---

I am currently a researcher at [JD.com](http://JD.com). I obtained my Ph.D. degree from the Chinese Academy of Sciences, Institute of Automation, supervised by [Prof. Jian Cheng](https://clab.ia.ac.cn/en), and the B.Eng. Degree from North China Electric Power University.

My research focuses on Data-centric AI and AI Infrastructure, with a specific emphasis on data mixture optimization, data selection, and data synthesis to construct a self-evolving data flywheel, as well as efficient model training and inference. These techniques have been successfully deployed in production within JD Retail's commodity understanding, yielding substantial business impact.


selected_papers: true # includes a list of papers marked as "selected={true}"
social: true # includes social icons at the bottom of the page

announcements:
  enabled: true # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: true
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---

I am currently a researcher at JD.com. I obtained my Ph.D. degree from the Chinese Academy of Sciences, Institute of Automation, supervised by [Prof. Jian Cheng](https://people.ucas.ac.cn/~chengjian?language=en), and the B.Eng. Degree from North China Electric Power University.

My research focuses on Data-centric AI and AI Infrastructure, with a specific emphasis on data mixture optimization, data selection, and data synthesis to construct a self-evolving data flywheel, as well as efficient model training and inference. These techniques have been successfully deployed in production within JD Retail's commodity understanding, yielding substantial business impact.



<!-- Put your address / P.O. box / other info right below your picture. You can also disable any of these elements by editing `profile` property of the YAML header of your `_pages/about.md`. Edit `_bibliography/papers.bib` and Jekyll will render your [publications page](/al-folio/publications/) automatically. -->

<!-- Link to your social media connections, too. This theme is set up to use [Font Awesome icons](https://fontawesome.com/) and [Academicons](https://jpswalsh.github.io/academicons/), like the ones below. Add your Facebook, Twitter, LinkedIn, Google Scholar, or just disable all of them.-->
