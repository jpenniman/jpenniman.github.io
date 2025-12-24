---
layout: post
title: Angular, Asprire, and Dev Containers
date: 2020-03-20
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: Let's take a look at how constants work in C# and why using static readonly might be better some cases.
tags:
- dotnet
- C#
---

ng serve by default binds to local host. To work in a dev container, we need to bind to the container ip:

``` js
  "scripts": {
    "ng": "ng",
    "start": "ng serve --host 0.0.0.0 --port 4200",
    "build": "ng build",
    "watch": "ng build --watch --configuration development",
    "test": "ng test"
  },

```

Devcontainer:

```js

```

