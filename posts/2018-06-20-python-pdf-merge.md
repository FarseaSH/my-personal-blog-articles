---
tags:
  - Python
category: 经验
toc: true
showupdate: false
copyright: true
date: 2018-06-20 11:47:37
title: 一个用于合并pdf的简单Python脚本
---
在学校打印店，有时会打印很多文件，因为文件数量多，过程会比较繁琐。自己没事动手写了一个pdf合并的python脚本，方便将多个pdf文件合并为一。这样打印时候只需点开一个文件打印即可。 

<!--more-->

## 使用方法 

### 必要的安装

需要[Python](https://www.python.org/)和[PyPDF2](https://github.com/mstamy2/PyPDF2)。Python安装可在官网找到；PyPDF2可以通过pip安装，可以利用命令行工具输入一下命令

```
pip install PyPDF2
```

### 使用

将需要合并的文件与本文后面的Python脚本放在同一目录下，运行脚本得到`Merged.pdf`即为合并的pdf文件。

{{<info>}}
脚本会将所在目录下所有pdf文件进行合并，须确保目录下只有需要合并的pdf文件。
{{</info>}}

## 使用Tips

### 按顺序合并pdf文件

如果需要按照一定顺序合并pdf文件，可以将pdf文件重命名，按顺序将文件重命名为`1.pdf`、`2.pdf`以此类推。

### 重复合并同一pdf文件

如果需要将某一pdf文件在合并文件中重复多次，可以将该文件直接在当前目录下拷贝成多个副本。

## 脚本代码

```python
"""
A simple python script to merge all the pdf files in the directory where this script is located.

@author: C.D.
"""

import PyPDF2
import os
import re


def main():
    # find all the pdf files in current directory.
    mypath = os.getcwd()
    pattern = r"\.pdf$"
    file_names_lst = [mypath + "\\" + f for f in os.listdir(mypath) if re.search(pattern, f, re.IGNORECASE) 
    and not re.search(r'Merged.pdf',f)]

    # merge the file.
    opened_file = [open(file_name,'rb') for file_name in file_names_lst]
    pdfFM = PyPDF2.PdfFileMerger()
    for file in opened_file:
        pdfFM.append(file)

    # output the file.
    with open(mypath + "\\Merged.pdf", 'wb') as write_out_file:
        pdfFM.write(write_out_file)

    # close all the input files.
    for file in opened_file:
        file.close()

if __name__ == '__main__':
    main()
```