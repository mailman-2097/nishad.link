# 🛋️ My personal blog 

This is my personal technical blog. It has the following features:

1. Dark and Light Theme Switcher
1. Search
1. Automated Table of Contents
1. Mobile device friendly
1. Blog Tags

![Typescript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)
![GitHub](https://img.shields.io/github/license/satnaing/astro-paper?color=%232F3741&style=for-the-badge)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-%23FE5196?logo=conventionalcommits&logoColor=white&style=for-the-badge)](https://conventionalcommits.org)
[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg?style=for-the-badge)](http://commitizen.github.io/cz-cli/)

## 🚀 Operational Guides

### Creating blog posts

Astro looks for `.astro` or `.md` files in the `src/pages/` directory. 

Each page is exposed as a route based on its file name.

Any static assets, like images, can be placed in the `public/` directory.

All blog posts are stored in `src/content/blog` directory.

### Publishing posts

This blog utlises a complete Gitops lifecycle.

Create a feature branch `blog/n/blogslug`

Review your changes by using a spell checker and preview your markdown script

Push your code and create a pull or merge request. 

A pipeline will run and give you a staging url to review your changes.

Once QA is complete; merge the pull request.

_Take care to `squash and merge` the pull request. Also, delete the feature branch when you are done._

#### Working with Dates

The template requires date field to be provided in the ISO 8601 fmt

```bash

# current timestamp in ISO 8601 format using Bash
TZ="Australia/Sydney" date -u +"%Y-%m-%dT%H:%M:%SZ"

```

#### Working with Images

By convention, images are saved in `png` format and favicon as `svg` format.

It is best to use the [vtracer](https://github.com/visioncortex/vtracer) tool to convert images.

```bash
# Ubuntu

sudo apt update -y
sudo apt  install cargo -y
cargo install vtracer

# Add the follwowing to your bash profile
# Add Cargo's bin directory to PATH
if [ -d "$HOME/.cargo/bin" ]; then
    PATH="$HOME/.cargo/bin:$PATH"
fi

vtracer --input blog3.png --output blog3.svg

```

## 💻 Tech Stack

**Main Framework** - [Astro](https://astro.build/)

**Website Theme** - [AstroPaper](https://astro.build/themes/details/astro-paper/)

**Web Hosting** - [Cloudflare](https://www.cloudflare.com/)

## ✨ Get in touch

You can contact me via my socials: 

* [Linkedin](www.linkedin.com/in/nishadsaithaly)
* [Mastadon](https://mastodon.social/@metaaverse)
* [BlueSky](https://bsky.app/profile/metaaverse0.bsky.social)

## 📜 License

Licensed under the MIT License, Copyright © 2025

---

Made with 🤍 by [Nishad K S](https://nishad.link)
