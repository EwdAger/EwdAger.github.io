---
title: Python读取Excel文件sheet名性能优化
tags:
  - Python
  - 性能优化
abbrlink: 6b56ef63
categories: 机器学习笔记
date: 2019-10-10 15:34:00
---

## 原始版本

直接使用`pandas`读取整个Excel文件，再从中取列名。这种场景对于小的Excel文件还适用，但数据量上升到10M+时，取个`sheet name`要`26s`之久。几乎无法忍受。

```python
data = pandas.ExcelFile(file_url)
names = data.sheet_names
```

## 优化

查阅资料可知`.xlsx`文件是一个压缩格式的文件，可以直接通过`zipfile`读到`sheet name`等相关信息。所以写如下函数直接取`sheet name`

<!--more-->

```python
def get_sheet_details(file_path):
    sheets = []
    file_name = os.path.splitext(os.path.split(file_path)[-1])[0]
    # 用文件名创建一个临时目录
    directory_to_extract_to = file_name
    os.mkdir(directory_to_extract_to)

    # 提取xlsx文件，因为它只是一个zip文件
    zip_ref = zipfile.ZipFile(file_path, 'r')
    zip_ref.extractall(directory_to_extract_to)
    zip_ref.close()

    # 建立一个临时的workbook文件
    path_to_workbook = os.path.join(directory_to_extract_to, 'xl', 'workbook.xml')
    with open(path_to_workbook, 'r') as f:
        xml = f.read()
        dictionary = xmltodict.parse(xml)
        # 多个sheet
        if isinstance(dictionary['workbook']['sheets']['sheet'], list):
            for sheet in dictionary['workbook']['sheets']['sheet']:
                # 有些版本的sheet是@name有些是@sheetId
                if sheet["@name"]:
                    meta_sheet = sheet["@name"]
                else:
                    meta_sheet = sheet["@sheetId"]
                sheets.append(meta_sheet)
        # 单个sheet
        else:
            sheet_dict = dictionary['workbook']['sheets']['sheet']
            if sheet_dict["@name"]:
                sheets.append(sheet_dict["@name"])
            else:
                sheets.append(sheet_dict["@sheetId"])

    shutil.rmtree(directory_to_extract_to)
    f.close()
    return sheets
```

## 一个问题
该函数只能针对`.xlsx`文件进行解析，而低版本的`.xls`文件就直接报错了，因为`.xls`是一个二进制文件而不是压缩文件。所以要以另一种方式去解析`sheet name`。经过查阅相关资料，发现`xlrd.open_workbook`的`on_demand=True`针对低版本的Excel文件可以只取列名而不加载数据。

```python
if file_url[-3:] == 'xls':
    names = xlrd.open_workbook(file_url, on_demand=True).sheet_names()
else:
    names = get_sheet_details(file_url)
```