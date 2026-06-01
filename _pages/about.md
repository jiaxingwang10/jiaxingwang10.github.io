---
layout: about
title: about
permalink: /
# 注意：subtitle 这里的引号请直接删掉，解决你上个截图里的“显眼包”问题
subtitle: Researcher, JD.com | Beijing, China | wjiaxing94@gmail.com

profile:
  align: left         # 👈 保持靠左
  image: prof_pic.jpg
  image_circular: false
  # 如果你不想要照片下面显示地址，可以把 address 这几行删了，或者留空
  address: 

# 下面这段代码是实现“右边对齐”的灵魂
style: |
  /* 1. 让个人简介部分变成“并排”布局 */
  .profile {
    float: left;
    width: 200px !important;   /* 👈 调整照片宽度 */
    margin-top: 0 !important;   /* 👈 强制顶格 */
    margin-right: 30px !important;
  }

  /* 2. 让名字和副标题往右顶，不被照片压在下面 */
  .post-header {
    display: block;
    overflow: hidden;         /* 👈 开启 BFC，让文字避开浮动的图片 */
    margin-top: 0 !important;
    padding-top: 0 !important;
  }

  header h1 {
    margin-top: 0 !important;  /* 👈 名字顶格 */
    font-size: 2.2rem;         /* 👈 名字可以稍微大一点点 */
  }

  /* 3. 手机端自动恢复堆叠，防止太挤 */
  @media (max-width: 576px) {
    .profile {
      float: none;
      width: 100% !important;
      margin: 0 auto 20px auto !important;
    }
  }
---


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
