---
layout: post
title: Removing custom Squarespace Layouts
date: 2023-02-27 12:17 -0500
tags:
  - Squarespace
---

When developing websites using the [Squarespace Developer Platform](https://developers.squarespace.com), one of the fundamental building blocks of each page is the site [layout](https://developers.squarespace.com/layouts-regions). The render chain of all pages starts with a layout, and they wrap the individual page contents. Generally the layout is reponsible for things like the `<html>` and `<body>` tags, global navs, etc.

Every page on the site has a selected layout (`Page Settings > Advanced > Page Layout`), and a site can contain any number of layouts.

If you choose to remove a layout (i.e., delete it from `layouts` in `template.conf`), but there are still pages using that layout, you may wonder what will happen. I haven’t found any documentation describing the intended behavior, but as best I can tell:

- The pages will continue to exist.
- They will be rendered using one of the remaining layouts, but it’s not clear how that gets decided (it may be the first layout defined in `tomplate.conf`).
- If you restore the layout to `template.conf`, the pages will begin using that layout again (i.e., the association seems to be keyed on the layout `name`).
- The `Page Layout` setting for the pages, at least in the UI, will be null, but can be switched to any remaining layout.
