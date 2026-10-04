---
date: 2023-08-05T22:36:00+08:00
title: PyWebIO 文本输入输出模版
category: 想法与笔记
tags: 
 - Python
 - 工具
toc: false
toc_fold: false
updated: 
copyright: true  # 暂不支持
draft: false
---

PyWebIO 是一个python框架，可以轻松为程序创建一个网页交互页面。我经常用python写一些输入文本返回文本结果的脚本，例如：输入一个长的json字符串，通过python函数返回一个格式化带缩进的json文本；又或是输入一个时间戳，返回一个中文年月日时间文本。这些脚本都可以通过PyWebIO建立起对应的网页交互界面。自己花了一点时间，面向此类文本输入输出任务，开发了一个简单的一个Pywebio页面模板。

![模版界面](https://s1.ax1x.com/2023/09/05/pPsnliq.png)

<!--more-->

主要开发了两个方便的小功能：

1. 支持实时 / 非实时结果输出两种模式：实时输出模式会在输入框有文本变动后，立即计算输出结果，适合计算量较轻的任务；非实时输出模式，需要单击转换按钮后才输出结果，适合计算量较大的任务或者需要调用其它外部服务的任务。
2. 支持一键拷贝输出结果

整个代码如下，将其中的`dojob`函数替换对应自己的Python函数即可，此外还可以修改`put_markdown("# 标题可替换").style("text-align: center")` 这段代码来自定义所想展示的标题。

```python
from pywebio.output import *
from pywebio.pin import *
from pywebio import *

import pyperclip

class AutoSwitch:
    def __init__(self) -> None:
        self.on = False  # Set default to on or off
    
    def on_click(self):
        self.on = not self.on
        self.put_ui()

    def put_ui(self):
        clear("auto_transform_switch")
        _button_label = "开" if self.on else "关"
        _button_color = "success" if self.on else "info"
        put_button(_button_label, color=_button_color, onclick=self.on_click, scope="auto_transform_switch")

auto_switch = AutoSwitch()

def _update_output(input_text="", load_from_input=False):
    if load_from_input: pin.pin_update("output", value=dojob(pin.pin['input']))
    else: pin.pin_update("output", value=dojob(input_text))

def manual_updat_output():
    _update_output(load_from_input=True)

def auto_update_output_when_on(input_text):
    if auto_switch.on: _update_output(input_text)

def copy_output():
    pyperclip.copy(pin.pin['output'])
    put_text("Copied!")

def dojob(input_text):
    """
    CHANGE HERE 转换的函数放在这里
    """
    return input_text

def main():
    put_markdown("# 标题可替换").style("text-align: center")

    put_row([
        put_text("自动转换状态").style("text-align: right; margin-right: 20px; font-size: 20px"),
        put_scope("auto_transform_switch")
    ])
    auto_switch.put_ui()

    put_row([
        put_textarea(name="input", rows=20, scope="input_area"),
        put_textarea(name="output", rows=20, scope="output_area"),
    ])
    put_row([
        put_button("转换", color="light", onclick=manual_updat_output).style("display: flex; justify-content: flex-end;"),
        put_button("拷贝", color='light', onclick=copy_output)
    ])

    pin.pin_on_change("input", onchange=auto_update_output_when_on)

if __name__ == "__main__":
    start_server(main, port=7777, debug=True)
```
