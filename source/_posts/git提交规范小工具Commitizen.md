---
title: git提交规范小工具Commitizen
date: 2018-04-23 16:08:15
tags: git
categories: tools
---

安装步骤

Conventional commit messages as a global utility
Install commitizen globally, if you have not already.

> npm install -g commitizen

Install your preferred commitizen adapter globally, for example cz-conventional-changelog

> npm install -g cz-conventional-changelog

Create a .czrc file in your home directory, with path referring to the preferred, globally installed, commitizen adapter

> echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc

<!--more-->

````bash
➜  Dresshelper git:(liu-master) ✗ git cz
cz-cli@2.8.6, cz-conventional-changelog@1.2.0

Line 1 will be cropped at 100 characters. All other lines will be wrapped after 100 characters.

? Select the type of change that you're committing: (Use arrow keys)
❯ feat:     A new feature
  fix:      A bug fix
  docs:     Documentation only changes
  style:    Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc.)
  refactor: A code change that neither fixes a bug or adds a feature
  perf:     A code change that improves performance
  test:     Adding missing tests
  chore:    Changes to the build process or auxiliary tools and libraries such as documentation generation
  revert:   Reverts a previous commit
````
