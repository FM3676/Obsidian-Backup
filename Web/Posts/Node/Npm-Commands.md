---
title: 'NPM Commonly used Commands'
date: '2022-7-28'
tags: ['Node','Tutorial','Snippets']
summary: 'Some commonly used Npm commands'
layout: PostSimple
---

- #### npm list ---> Show all the dependencies version.

- #### npm list --depth=0 --> Only show the dependencies of your application.

- #### npm view -packageName -> See the specific module’s package.json.

- #### npm view -packageName dependencies --> See the specific module’s dependencies.

- #### npm view -packageName version --> See the specific module’s all version.

- #### npm view -packageName property --> See the specific module’s property.(Any property in the package.json, such as dependencies).

- #### npm outdated --> See what package have been updated and what are the new version

- #### npm update --> Update the module, only works for updating minor and patch released.

- #### npm i -packageName --save-dev --> Install the module in develop environment.

- #### npm uninstall -packageName || npm un  -packageName --> Uninstall the module.

- #### npm i -g  -packageName --> Install global package.

- #### npm -g outdated --> Check out all the global package installed.


