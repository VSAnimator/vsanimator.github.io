# Blog Structure

This Jekyll site now includes a blog section with the following structure:

## Files Created

### Layouts
- `_layouts/post.html` - Layout for individual blog posts
- `_layouts/blog.html` - Layout for the blog index page

### Pages
- `blog.markdown` - Blog index page (accessible at `/blog/`)
- `about.md` - About page (accessible at `/about/`)

### Posts
- `_posts/2025-01-15-welcome-to-my-blog.md` - Test blog post

### Styling
- `assets/blog.css` - Custom CSS for blog styling

## How to Add New Blog Posts

1. Create a new markdown file in the `_posts/` directory
2. Use the filename format: `YYYY-MM-DD-title.md`
3. Include the following front matter at the top:

```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD
categories: [category1, category2]
tags: [tag1, tag2]
---
```

4. Write your content in markdown format
5. The post will automatically appear on the blog index page

## Features

- **Automatic Post Listing**: All posts in `_posts/` are automatically listed on the blog page
- **Date Formatting**: Posts show formatted dates
- **Categories and Tags**: Support for organizing posts
- **MathJax Support**: Mathematical expressions are supported
- **Responsive Design**: Blog works on mobile and desktop

## Navigation

The blog is accessible via the "Blog" link in the site navigation. The navigation is automatically generated from the `header_pages` list in `_config.yml`.

## Customization

- Edit `assets/blog.css` to customize the blog styling
- Modify `_layouts/post.html` or `_layouts/blog.html` to change the layout
 