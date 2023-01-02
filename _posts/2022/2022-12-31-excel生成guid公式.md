---
title: "excel生成guid公式"
publishDate: 2022-12-31 00:26:00 +0800
date: 2022-12-31 00:14:08 +0800
categories: windows excel
position: problem
---

excel生成guid公式

---

<div id="toc"></div>

## 大写

`=CONCATENATE(DEC2HEX(RANDBETWEEN(0,4294967295),8),"-",DEC2HEX(RANDBETWEEN(0,65535),4),"-",DEC2HEX(RANDBETWEEN(0,65535),4),"-",DEC2HEX(RANDBETWEEN(0,65535),4),"-",DEC2HEX(RANDBETWEEN(0,4294967295),8),DEC2HEX(RANDBETWEEN(0,65535),4))`

## 小写

`=LOWER(CONCATENATE(DEC2HEX(RANDBETWEEN(0,4294967295),8),"-",DEC2HEX(RANDBETWEEN(0,65535),4),"-",DEC2HEX(RANDBETWEEN(0,65535),4),"-",DEC2HEX(RANDBETWEEN(0,65535),4),"-",DEC2HEX(RANDBETWEEN(0,4294967295),8),DEC2HEX(RANDBETWEEN(0,65535),4)))`

---

**参考资料**

- [How to create a GUID in Excel?](https://stackoverflow.com/questions/14990236/how-to-create-a-guid-in-excel)
