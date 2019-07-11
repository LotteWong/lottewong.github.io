---
title: \#Configuration\# 从零到壹：GitHub Pages + Hexo = Blog
categories: Configuration
tags:
- GitHub Pages
- Hexo
thumbnail: /images/hexo-github.png
---



利用**GitHub Pages+Hexo**打造一个个人博客，主要分为以下**五个部分**：

- 环境准备 Environment
- **文件配置 Configuration**
- 个性化 Customization
- **博客写作 Writing**
- 双备份 Backup



<!-- more -->



## **环境准备 Environment** 

### **安装Git + Github**

- **安装Git部署插件:** 

  ```powershell
  $ npm install hexo-deployer-git --save
  ```

### **安装Node.js**

- **安装Node.js:** 

  [Download | Node.js](https://link.zhihu.com/?target=https%3A//nodejs.org/en/download/)

- **检查是否安装成功:**

  ```powershell
  $ node -v
  $ npm -v
  ```

### **安装Hexo**

- **安装Hexo:**

  ```powershell
  $ npm install -g hexo-cli
  ```

- **检查是否安装成功:**

  ```powershell
  $ hexo -v
  ```

- **初始化:**

  ```powershell
  $ hexo init blog
  ```

## **文件配置 Configuration**

### **本地运行**

```powershell
$ hexo clean # 删除缓存
$ hexo g # 生成Hexo页面
$ hexo s # 本地部署Hexo页面
```

### **远程运行**

```powershell
$ hexo clean # 删除缓存
$ hexo g # 生成Hexo页面
$ hexo d # 远程部署Hexo页面
```

### **基本配置**

**/_config.yml**

```yaml
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: LotteWong # 个人博客显示名称
subtitle: 在代码符号表象中避难。 # 个人博客副标题
description: # 搜索引擎描述信息
keywords: # 搜索引擎关键词
author: LotteWong # 网站作者
avatar: ./themes/icarus/source/images/favicon.ico # 网站头像
language: en # 网站语言
timezone: Asia/HongKong # 网站时区

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
  
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date
  
# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
plugins: # 设置个人博客插件
theme: icarus # 设置个人博客主题

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy: 
  type: git # 部署类型
  repo: git@github.com:LotteWong/lottewong.github.io.git # 部署仓库
  branch: master # 部署分支
```

## **个性化 Customization**

**/themes/icarus/_config.yml**

```yaml
# Version of the Icarus theme that is currently used
version: 2.3.0
# Path or URL to the website's icon
favicon: /images/favicon.ico
# Path or URL to RSS atom.xml
rss: 
# Path or URL to the website's logo to be shown on the left of the navigation bar or footer
logo: /images/logo.png
# Open Graph metadata
# https://hexo.io/docs/helpers.html#open-graph
open_graph:
    # Facebook App ID
    # fb_app_id: 
    # Facebook Admin ID
    # fb_admins: 
    # Twitter ID
    # twitter_id: 
    # Twitter site
    # twitter_site: 
    # Google+ profile link
    # google_plus: 
# Navigation bar link settings
navbar:
    # Navigation bar menu links
    menu:
        Home: /
        Archives: /archives
        Categories: /categories
        Tags: /tags
        About: /about
    # Navigation bar links to be shown on the right
    # links:
        # Download on GitHub:
            # icon: fab fa-github
            # url: 'https://github.com/LotteWong'
# Footer section link settings
footer:
    # Links to be shown on the right of the footer section
    links:
        Github:
            icon: fab fa-github
            url: 'https://github.com/LotteWong'
        RSS:
            icon: fas fa-rss
            url: 'https://www.zhihu.com/people/lai-xiu-ping'
# Article display settings
article:
    # Code highlight theme
    # https://github.com/highlightjs/highlight.js/tree/master/src/styles
    highlight: atom-one-light
    # Whether to show article thumbnail images
    thumbnail: true
    # Whether to show estimate article reading time
    readtime: true
# Search plugin settings
# https://ppoffice.github.io/hexo-theme-icarus/categories/Plugins/Search
search:
    # Name of the search plugin
    type: insight
# Comment plugin settings
# https://ppoffice.github.io/hexo-theme-icarus/categories/Plugins/Comment
comment:
    # Name of the comment plugin
    type: 
# Donation entries
# https://ppoffice.github.io/hexo-theme-icarus/categories/Donation/
donate:
    -
        # Donation entry name
        type: alipay
        # Qrcode image URL
        qrcode: '/images/alipay.png'
    -
        # Donation entry name
        type: wechat
        # Qrcode image URL
        qrcode: '/images/wechat.png'
    -
        # Donation entry name
        type: paypal
        # Paypal business ID or email address
        business: 'SuperGsama@outlook.com'
        # Currency code
        currency_code: USD
    -
        # Donation entry name
        # type: patreon
        # URL to the Patreon page
        # url: ''
# Share plugin settings
# https://ppoffice.github.io/hexo-theme-icarus/categories/Plugins/Share
share:
    # Share plugin name
    type: 
# Sidebar settings.
# Please be noted that a sidebar is only visible when it has at least one widget
sidebar:
    # left sidebar settings
    left:
        # Whether the left sidebar is sticky when page scrolls
        # https://ppoffice.github.io/hexo-theme-icarus/Configuration/Theme/make-a-sidebar-sticky-when-page-scrolls/
        sticky: false
    # right sidebar settings
    right:
        # Whether the right sidebar is sticky when page scrolls
        # https://ppoffice.github.io/hexo-theme-icarus/Configuration/Theme/make-a-sidebar-sticky-when-page-scrolls/
        sticky: false
# Sidebar widget settings
# https://ppoffice.github.io/hexo-theme-icarus/categories/Widgets/
widgets:
    -
        # Widget name
        type: profile
        # Where should the widget be placed, left or right
        position: left
        # Author name to be shown in the profile widget
        author: LotteWong
        # Title of the author to be shown in the profile widget
        author_title: SCUT, Undergraduate
        # Author's current location to be shown in the profile widget
        location: Guangzhou, China
        # Path or URL to the avatar to be shown in the profile widget
        avatar:
        # Email address for the Gravatar to be shown in the profile widget
        gravatar: 
        # Whether to show avatar image rounded or square
        avatar_rounded: false
        # Path or URL for the follow button
        follow_link: 'https://www.jianshu.com/u/80ee6b6f3418'
        # Links to be shown on the bottom of the profile widget
        social_links:
            Project:
                icon: fab fa-creative-commons
                url: 'https://github.com/scutse-man-month-myth/InkYear'
            Organization:
                icon: fab fa-creative-commons-by
                url: 'https://github.com/scutse-man-month-myth'
            Developer:
                icon: fab fa-github
                url: 'https://github.com/LotteWong'
            #Facebook:
                #icon: fab fa-facebook
                #url: 'https://facebook.com'
            #Twitter:
                #icon: fab fa-twitter
                #url: 'https://twitter.com'
            #Dribbble:
                #icon: fab fa-dribbble
                #url: 'https://dribbble.com'
    -
        # Widget name
        type: toc
        # Where should the widget be placed, left or right
        position: left
    -
        # Widget name
        type: links
        # Where should the widget be placed, left or right
        position: left
        # Links to be shown in the links widget
        links:
            Dart: 'https://dart.dev/'
            Flutter: 'https://flutter.dev/'
    -
        # Widget name
        type: category
        # Where should the widget be placed, left or right
        position: left
    -
        # Widget name
        type: tagcloud
        # Where should the widget be placed, left or right
        position: left
    -
        # Widget name
        type: recent_posts
        # Where should the widget be placed, left or right
        position: right
    -
        # Widget name
        type: archive
        # Where should the widget be placed, left or right
        position: right
    -
        # Widget name
        type: tag
        # Where should the widget be placed, left or right
        position: right
# Other plugin settings
plugins:
    # Enable page animations
    animejs: true
    # Enable the lightGallery and Justified Gallery plugins
    # https://ppoffice.github.io/hexo-theme-icarus/Plugins/General/gallery-plugin/
    gallery: true
    # Enable the Outdated Browser plugin
    # http://outdatedbrowser.com/
    outdated-browser: true
    # Enable the MathJax plugin
    # https://ppoffice.github.io/hexo-theme-icarus/Plugins/General/mathjax-plugin/
    mathjax: true
    # Show the back to top button on mobile devices
    back-to-top: true
    # Google Analytics plugin settings
    # https://ppoffice.github.io/hexo-theme-icarus/Plugins/General/site-analytics-plugin/#Google-Analytics
    google-analytics:
        # Google Analytics tracking id
        tracking_id: 
    # Baidu Analytics plugin settings
    # https://ppoffice.github.io/hexo-theme-icarus/Plugins/General/site-analytics-plugin/#Baidu-Analytics
    baidu-analytics:
        # Baidu Analytics tracking id
        tracking_id: 
    # Hotjar user feedback plugin
    # https://ppoffice.github.io/hexo-theme-icarus/Plugins/General/site-analytics-plugin/#Hotjar
    hotjar:
        # Hotjar site id
        site_id: 
    # Show a loading progress bar at top of the page
    progressbar: true
    # Show the copy button in the highlighted code area
    clipboard: true
    # BuSuanZi site/page view counter
    # https://busuanzi.ibruce.info
    busuanzi: false
# CDN provider settings
# https://ppoffice.github.io/hexo-theme-icarus/Configuration/Theme/speed-up-your-site-with-custom-cdn/
providers:
    # Name or URL of the JavaScript and/or stylesheet CDN provider
    cdn: jsdelivr
    # Name or URL of the webfont CDN provider
    fontcdn: google
    # Name or URL of the webfont Icon CDN provider
    iconcdn: fontawesome
```

## **博客写作 Writing**

- **默认**

  ```powershell
  $ hexo new "blog title"
  ```

- **自定义**

  ```yml
  title: {{ blog title }}
  categories: {{ blog category }}
  tags:
  - {{ blog tag }}
  thumbnail: {{ blog thumbnail }}
  
  {{ Abstract }}
  
  <!-- more -->
  
  {{ Content }}
  ```

## **双备份 Backup**

- **Hexo备份:** 

  ```powershell
  # master branch
  $ hexo d
  ```

- **Src备份:** 

  ```powershell
  # dev branch
  $ git checkout dev
  $ git add --all
  $ git commit -m "new blog"
  $ git push origin dev
  ```

## **待办事项 Todos**

- 对应图标
- 更多插件
- 绑定域名
- 更新外链

## **参考链接 References**

- [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
- [Hexo](https://github.com/hexojs/hexo/wiki/Themes)
- [icarus](https://blog.zhangruipeng.me/hexo-theme-icarus/categories/)