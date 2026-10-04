---
date: 2022-04-30T21:03:00+08:00
title: 利用Github Action定时运行Python脚本
category: 想法与笔记

tags: 
 - Python
 - Github Action

toc: false
toc_fold: false

updated: 
copyright: true  # 暂不支持

draft: false
---

自己业余时间经常会开发一些脚本，定时运行它们，来辅助自己完成各种各样的任务，比如：定时推送天气情况到自己的todo list；定时运行一些爬虫，摘录新闻、消息（例如[优化热榜类RSS feed源](https://www.yyshao.com/2022/04/09/trending_rss_feed/)）；定期整理自己的财务数据，输出报表等等等等。

以前我用过云服务设置cron job定时来跑这些脚本，但是云服务的使用是需要付费的。后来Github Action发布，public repo可以无限制使用，private repo也提供一定的免费时长额度。Action job可以通过cron形式触发，我们可以借此实现定时运行Python脚本。整个实现过程只需要两步，本文简单记录一下过程。

<!--more-->

以下是我定时任务repo的目录结构，可供参考

```
root
├── .github
│   └── workflows
│       └── workflow_config.yml
└── src
    └── your_python_file.py
    
```

其中，`.github`下`workflows`目录下存放github action相关文件，最关键的`workflow_config.yml`用于action的配置

```yaml
name: Your Github Workflow name
on:
  workflow_dispatch:
  schedule:
    # IMPORTANT: Set cron job in UTC timezone
    - cron:  '0 18 * * *'

jobs:
  run-python-script:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
        with:
          python-version: '3.8.0'
      - run: pip install requests
      - run: python src/first_python_script.py
      
```

几点说明的是：

1. cron job的时间应该用UTC时区的时间，而**非**国内用的东八区时间（北京时间）。
2. 可以用[crontab.guru](https://crontab.guru/)这个网站辅助我们写cron表达式。
3. 添加`run: pip install requests`这一行指令，来安装配置所需要依赖的环境。

