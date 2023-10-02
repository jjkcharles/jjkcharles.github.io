---
layout: post
title: Jekyll - Remote themes
subtitle: Always ask for the specific version!
author: jjk_charles
categories: tips
tags: blog jekyll  
---

When I first started this blog using Jekyll, I wasn't really paying close attention to how [remote themes plugin](https://github.com/benbalter/jekyll-remote-theme) work. I guess I was more interested in getting it up and running, and I probably didn't care as long as things worked.

I was exploring the recently released [TypeChat library](https://microsoft.github.io/TypeChat/blog/introducing-typechat/) by Microsoft and wanted to get a draft blog post going for it, even though it's not yet ready to be published. 

While I had written the first few lines of the post, as usual, I wanted to run a quick build to see if everything looked fine with the new page. This is when I noticed that the blog looked a little different than usual, and few things seemed to have broken as I wasn't using thumbnail images for my posts.

After a little digging around, I realized that the remote theme I was using had released an update that was causing this issue. I initially tried to figure out what new features had been released and see if there was a way to turn them OFF - at least the ones that were causing the issue.

30 minutes later, I had made no progress at all. That's when I found out that Jekyll does allow [referring to a specific version of the theme](https://github.com/benbalter/jekyll-remote-theme#declaring-your-theme) rather than the latest version.

The fix turned out to be stupidly simple - since Jekyll's remote themes use Github repositories as the theme source, all I had to do was reference the commit ID (or) branch/tag name along with the repo name,

```yaml
#theme: jekyll-theme-yat
remote_theme: "jeffreytse/jekyll-theme-yat@3c6242f"
```