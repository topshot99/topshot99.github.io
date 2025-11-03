# TopShot99's Tech Blog

A Jekyll-powered blog for sharing technical insights, discoveries, and learnings in software development.

## 🚀 Live Site

Visit the blog at: [https://topshot99.github.io](https://topshot99.github.io)

## 📝 About

This blog focuses on:
- Azure Cosmos DB and database technologies
- JavaScript/TypeScript development
- Performance optimization and debugging
- Cloud development patterns
- Real-world problem-solving experiences

## 🛠️ Local Development

### Prerequisites
- Ruby (3.1 or higher)
- Bundler

### Setup
```bash
# Install dependencies
bundle install

# Serve locally
bundle exec jekyll serve

# Open http://localhost:4000 in your browser
```

### Adding New Posts

Create a new file in `_posts/` with the format: `YYYY-MM-DD-title.md`

Example frontmatter:
```yaml
---
layout: post
title: "Your Post Title"
date: 2025-11-03 14:30:00 +0000
categories: [category1, category2]
tags: [tag1, tag2, tag3]
author: TopShot99
excerpt: "Brief description of your post"
---
```

## 📁 Project Structure

```
├── _config.yml          # Site configuration
├── _posts/               # Blog posts
├── _layouts/             # Page layouts (if custom)
├── _includes/            # Reusable components (if custom)
├── _sass/                # Custom styles (if any)
├── assets/               # Images, CSS, JS
├── .github/workflows/    # GitHub Actions for deployment
├── index.md              # Homepage
├── about.md              # About page
└── Gemfile               # Ruby dependencies
```

## 🎨 Customization

This blog uses the Minima theme. You can customize:
- Site colors and fonts by overriding Sass variables
- Layout by creating custom layouts in `_layouts/`
- Add custom pages in the root directory

## 🚀 Deployment

The site automatically deploys to GitHub Pages via GitHub Actions when you push to the `main` branch.

### Setup Steps:
1. Create a repository named `topshot99.github.io` (or any name)
2. Push this code to the repository
3. Go to Settings > Pages
4. Set Source to "GitHub Actions"
5. The site will be available at `https://topshot99.github.io`

## 📊 Features

- **Responsive Design**: Mobile-friendly layout
- **Syntax Highlighting**: Code blocks with Rouge highlighter
- **SEO Optimized**: Meta tags, sitemap, and structured data
- **RSS Feed**: Automatic feed generation
- **Fast Loading**: Optimized for performance
- **GitHub Integration**: Easy deployment and version control

## 🤝 Contributing

Feel free to:
- Report issues or suggest improvements
- Submit pull requests for bug fixes
- Suggest new features or content ideas

## 📄 License

This blog is open source. Feel free to use it as a template for your own blog!

---

**Happy blogging!** 🎉