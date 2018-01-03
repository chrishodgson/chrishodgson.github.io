---
layout: post
title: Upgrade Hyde theme from Jekyll 2 to 3
excerpt: 
tags: [jekyll upgrade]
lastmod: 2017-11-23T07:44:30+00:00
links:
    - {link: "https://jekyllrb.com/docs/upgrading/2-to-3/", label: "Upgrading from 2.x to 3.x"}        
---

#### Jekyll 3 Install 
`gem install jekyll`

note: if this returns that it `requires > ruby 2.0.11` then 
`rvm install ruby-[latest]`

#### Changes to make Hyde theme compatible with jekyll 3

1. _config.yml  

    - add line: `paginate_path: "page:num"`
    - add line: `plugins: [jekyll-paginate, jeyll-gist]`
    - remove line: `relative_permalinks: true`

1. `gem install jekyll-paginate redcarpet jekyll-gist`

1. `jekyll serve && open http://127.0.0.1:4000`

