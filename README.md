# ComplyTime Website

The official website for [ComplyTime](https://github.com/complytime) - Cloud Native Compliance, Reimagined.

Built with [Hugo](https://gohugo.io/) and the [Doks](https://getdoks.org/) theme.

## 🚀 Quick Start

### Prerequisites

- [Node.js](https://nodejs.org/) v18 or later
- [npm](https://www.npmjs.com/) (included with Node.js)

### Development

```bash
# Install dependencies
npm install

# Start development server
npm run dev
```

The site will be available at `http://localhost:1313/`.

### Build

```bash
# Build for production
npm run build
```

The output will be in the `public/` directory.

## 📁 Project Structure

```
website/
├── assets/                # SCSS, JavaScript, images
│   └── scss/
│       └── common/
│           ├── _custom.scss
│           └── _variables-custom.scss
├── config/                # Hugo configuration
│   └── _default/
│       ├── hugo.toml
│       ├── params.toml
│       └── menus/
│           └── menus.en.toml
├── content/               # Markdown content
│   ├── _index.md          # Homepage
│   ├── docs/              # Documentation
│   │   ├── getting-started/
│   │   ├── projects/      # Project pages (complyctl, complyscribe, etc.)
│   │   ├── architecture/
│   │   ├── tutorials/     # Step-by-step guides (placeholder entries)
│   │   └── contributing/
│   └── privacy.md
├── layouts/               # Custom layouts
│   ├── home.html          # Homepage layout
│   └── docs/
│       └── list.html      # Docs section listing layout
├── static/                # Static assets (favicons, icons)
├── .github/
│   └── workflows/
│       └── deploy-gh-pages.yml  # CI/CD deployment
└── package.json
```

## 📝 Content

### Navigation

| Menu Item  | URL                    | Description                       |
|------------|------------------------|-----------------------------------|
| Docs       | `/docs/getting-started/` | Documentation landing page       |
| Projects   | `/docs/projects/`      | ComplyTime project pages          |
| Tutorials  | `/docs/tutorials/`     | Step-by-step guides               |
| Community  | `/docs/contributing/`  | Contribution and community info   |

### Adding Documentation

Create a new Markdown file in the appropriate directory under `content/docs/`:

```markdown
---
title: "Page Title"
description: "Page description"
lead: "Brief intro text"
date: 2024-01-01T00:00:00+00:00
draft: false
weight: 100
toc: true
---

Your content here...
```

### Adding Tutorials

Create a new Markdown file under `content/docs/tutorials/`:

```markdown
---
title: "Tutorial Title"
description: "What the reader will learn"
lead: "A brief summary of the tutorial."
date: 2024-01-01T00:00:00+00:00
draft: false
weight: 110
toc: true
---

## Overview
...

## Prerequisites
...

## Steps
...
```

## 🎨 Customization

### Styling

Custom styles are in `assets/scss/common/`:
- `_variables-custom.scss` - Variables and theme colors
- `_custom.scss` - Additional custom styles

### Configuration

Site configuration is in `config/_default/`:
- `hugo.toml` - Hugo settings
- `params.toml` - Theme parameters
- `menus/menus.en.toml` - Navigation menus

## 🚢 Deployment

The site is deployed to GitHub Pages via the `.github/workflows/deploy-gh-pages.yml` workflow. On push to the configured branch, GitHub Actions builds the site with Hugo and deploys the `public/` directory.

## 🤝 Contributing

Contributions are welcome! Please see our [Contributing Guide](https://github.com/complytime/community).

## 📄 License

This website is licensed under [Apache 2.0](LICENSE).

## 🔗 Links

- [ComplyTime GitHub](https://github.com/complytime)
- [Doks Theme](https://getdoks.org/)
- [Hugo](https://gohugo.io/)
