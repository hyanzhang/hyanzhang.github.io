---
layout: post
title: a post with jupyter notebook
date: 2023-07-04 08:57:00-0400
description: an example of a blog post with jupyter notebook
tags: formatting jupyter
categories: sample-posts
giscus_comments: false
related_posts: false
---
## B-Spline算法原理
[参考链接](https://rohangautam.github.io/blog/b_spline_intro/)
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/B-splines.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Cox-de Boor algorithm for B-spline basis functions.
</div>
## 代码实现：
{::nomarkdown}
{% assign jupyter_path = "assets/jupyter/blog.ipynb" | relative_url %}
{% capture notebook_exists %}{% file_exists assets/jupyter/blog.ipynb %}{% endcapture %}
{% if notebook_exists == "true" %}
{% jupyter_notebook jupyter_path %}
{% else %}

<p>Sorry, the notebook you are looking for does not exist.</p>
{% endif %}
{:/nomarkdown}

Note that the jupyter notebook supports both light and dark themes.
