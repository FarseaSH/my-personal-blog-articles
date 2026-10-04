---
title: Github Action 推送项目到 Vercel
date: 2023-11-28T20:51:00+08:00
updated:
copyright: true  # 暂不支持
category: 想法与笔记
tags:
    - Github Action
    - Vercel
toc: true
toc_fold: false
draft: false
---

最近在做一个小的项目，项目会生成静态网页，并通过Vercel进行部署发布。项目并未采用Vercel监听Github repo的commit变化，自动进行同步的方式，而是利用Github Action，通过Vercel CLI提交到Vercel，这篇笔记记录一下相关配置的细节，以方便后续其它项目部署使用。

<!--more-->

## Github Action配置

我们使用 `amondnet/vercel-action` Action来将项目文件同步至Vercel，在Github Action yaml配置文件中添加如下配置

```yaml
- uses: amondnet/vercel-action@v20
  with:
  vercel-token: ${{ secrets.VERCEL_TOKEN }}
  github-token: ${{ secrets.GITHUB_TOKEN }}
  vercel-args: '--prod'
  vercel-org-id: ${{ secrets.ORG_ID}}
  vercel-project-id: ${{ secrets.PROJECT_ID}}
  working-directory: ./public
```

其中`working-directory`字段需要改成相应推送文件所在的路径

## 环境变量的配置

在github repo setting中，选择 XXX/secret，依此加入3个环境变量

- `ORG_ID`
- `VERCER_TOKEN`
- `PROJECT_ID`

这三个变量都在Vercel中获取，具体获取方式如下

`ORG_ID`：在个人Setting页面中找到`Vercel ID`项，复制即可

![ORG_ID](https://s11.ax1x.com/2023/12/20/pioOFiT.png)

`VERCEL_TOKEN`：在个人Setting页面找到`Tokens`项，创建一个Token后会有一串字符，复制即可

![VERCEL_TOKEN](https://s11.ax1x.com/2023/12/20/pioO9Zq.png)

`PROJECT_ID`：在Project Setting页面中找到`General`栏下 `Project ID`项，复制即可

![PROJECT_ID](https://s11.ax1x.com/2023/12/20/pioLOJS.png)

其它批注

- Vercel创建项目时，如果为了部署静态网站，Template选择Other即可，无需更改其它配置
- 如果Github repo没有commit更新，一段时间后，Github Action 定时任务会自动暂停掉，需要手动激活