---
author: Nishad Saithaly
pubDatetime: 2023-03-07T02:28:59Z
title: Getting started with JAMstack
postSlug: getting-started-with-jamstack
featured: true
draft: false
tags:
  - jamstack
  - nodejs
  - javascript
  - typescript
  - markdown
  - astrojs
  - ssg
ogImage: "/assets/jamstack_logo.svg"
description: Humble beginnings of my blog
---

## Table of contents

## Introduction

This is my first blog built using Nodejs. Under the hood it uses [Astro](https://astro.build/).

Astro is a javascript or typescript based templating engine and runs on [nodejs](https://nodejs.org/en/).

## Static Site Generators

In my search for setting up a blog, I came across another tech acronym [JAMstack](https://jamstack.org/)
and it introduced me to the next generation of websites called Static Site Generators (SSG).

SSGs are essentially javascript sites that are compiled into static pages; all that means is _no database required_.

## Looking at the options

There are a lot of good SSGs out there are. You can find them on [JAMstack](https://jamstack.org/).

Here are my thoughts based on my research:

| S. No. | SSG                                    | Pros                                                                   | Cons                                              | Remarks                           |
| ------ | -------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------- | --------------------------------- |
| 1      | [Hugo CMS](https://gohugo.io/)         | - Lightweight<br>- Many themes<br>- Strong Community                   | - themes are git sub-modules                      | Its not hugo its me.              |
| 2      | [Jekyll](https://jekyllrb.com/)        | - Proper templating engine IMHO<br>- Many themes<br>- Strong Community | - Many theme sites                                | Better curation of Jekyll themes  |
| 3      | [Gatsby JS](https://www.gatsbyjs.com/) | - Very flexible<br>- Many themes<br>- Strong Community                 | - Need JS knowledge                               | Plan for v2 blog is to use gatsby |
| 4      | [Astro](https://astro.build/)          | - Meta framework<br>                                                   | - Few themes<br>- New entry<br>- Few integrations | Its not easy to customise         |

## Why Astro?

Astro happened to be the new kid on the block and it hit 2.0.

They have official themes on their website and the documentation was good.

Obviously, there is a learning curve and the number of integrations (can I say plugins) are growing.

From my experience as a Java developer, I know that I did not have the time to create a theme from scratch.

However, I found a few blog themes that had all the main functionality I needed:

1. Dark and Light mode
2. Search
3. Tags
4. etc..

So I picked one and here we are.

## Closing thoughts

I want a theme that is my own.

I did spend some time on [Gatsby JS](https://www.gatsbyjs.com/) so I am thinking my v2 may be gatsby based.

But that's for another post.
