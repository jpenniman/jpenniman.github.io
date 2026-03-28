---
layout: post
title: How I Setup Projects
date: 2026-03-15
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: 
tags:
- dotnet
- C#
---

# Introduction

I'm often asked how I set up projects and why, so I thought I'd write a bit about it. Obviously every project is
different, but many of the long term things I build end up in this structure or something similar.

# The Repository Root

Regardless of the type of project, at the root, you'll almost always find the trinity:

```
/
├─.editorconfig
├─.gitignore
└─README.md
```

If I'm using AI coding assistants, there will also be an `AGENTS.md` file.

```
/
├─.editorconfig
├─.gitignore
├─AGENTS.md
└─README.md
```

If it's an open source project, there will be a `LICENSE.md`. Or, if a commercial product, a `EULA.md`:

```
/
├─.editorconfig
├─.gitignore
├─AGENTS.md
├─LICENSE.md
└─README.md
```


```
/
├─docs/
├─src/
├─tests/
├─AGENTS.md
├─.editorconfig
├─.gitignore
├─Directory.Build.props
├─Directory.Packages.props
├─README.md
└─Solution.slnx
```

```
Branch Connector        ├   U+251C: BOX DRAWINGS LIGHT VERTICAL AND RIGHT
Leaf Connector          └   U+2514: BOX DRAWINGS LIGHT UP AND RIGHT
Horizontal Connector    ─   U+2500: BOX DRAWINGS LIGHT HORIZONTAL
Vertical Connector      │   U+2502: BOX DRAWINGS LIGHT VERTICAL

root/
├── ape/
├── bat/
├── cat/
│   ├── cat.html
│   ├── cat.md
│   └── cat.txt
├── dog/
│   ├── elf/
│   │   ├── elf.html
│   │   ├── elf.md
│   │   └── elf.txt
│   ├── dog.html
│   ├── dog.md
│   └── dog.txt
└── README.md

```
