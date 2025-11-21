# Louise's Musings

Personal blog and portfolio site built with Hugo and the Ananke theme.

## 🌐 Live Site

https://matcha-puddings.github.io/

## 📝 Content Sections

- **Tech Blog** - Technical articles about LLMs, RAG, AI agents, and engineering
- **Book Reviews** - Book summaries and insights on personal development
- **Photography** - Personal photography portfolio

## 🛠️ Local Development

```bash
# Install Hugo (if not already installed)
brew install hugo

# Clone the repository
git clone https://github.com/matcha-puddings/matcha-puddings.github.io.git
cd matcha-puddings.github.io

# Initialize theme submodule
git submodule update --init --recursive

# Run local development server
hugo server -D

# Site will be available at http://localhost:1313/
```

## 🚀 Deployment

This site automatically deploys to GitHub Pages via GitHub Actions whenever changes are pushed to the `main` branch.

The workflow:
1. Push changes to `main` branch
2. GitHub Actions builds the Hugo site
3. Deploys to https://matcha-puddings.github.io/

## 📦 Tech Stack

- **Static Site Generator**: Hugo v0.152.2
- **Theme**: [Ananke](https://github.com/theNewDynamic/gohugo-theme-ananke)
- **Hosting**: GitHub Pages
- **CI/CD**: GitHub Actions

## 📄 License

Content is © Louise Cheung. All rights reserved.
