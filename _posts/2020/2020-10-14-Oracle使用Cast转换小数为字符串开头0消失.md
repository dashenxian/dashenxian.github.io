---
title: "Oracle使用Cast转换小数为字符串开头0消失"
publishDate: 2020-10-14 19:26:00 +0800
date: 2020-10-14 19:14:08 +0800
categories: Oracle CSharp
position: problem
---

在oracle中需要把小数转换成字符串返回，这时如果你的数字只有小数部分有值如：0.5，使用cast或者to_char转换都会变成".5"，这是不符合中国人习惯的。

---

<div id="toc"></div>

在oracle中需要把小数转换成字符串返回，这时如果你的数字只有小数部分有值如：0.5，使用cast转换会变成".5"，这是不符合中国人习惯的。

```sql
SELECT CAST(0.5 as Varchar(20)) from dual;
```

这时如果你到网上搜索，不管你怎么搜都只会得到下面这样让你格式化字符串的答案：

```sql
SELECT to_char(0.5,'fm9999990.9999') from dual;
```

但是这样仍然有问题，就是你的数刚好是整数，没有小数点，这是会给你加上小数点。

```sql
SELECT to_char(1,'fm9999990.9999') from dual;
```

结果变成了这样：

```cmd
1.
```

一直冥思苦想都没找到办法解决。经过多方查证（在群里问了下），我的方向选错了，我一直想找一个原生的转换，完全忽略了可以针对这种情况单独处理一下，比如末尾有小数点时直接移除。

```sql
select RTRIM(to_char(0.5,'fm9999999990.9999'),'.')
,RTRIM(to_char(1,'fm9999999990.9999'),'.')
,RTRIM(to_char(1.5,'fm9999999990.9999'),'.')
from dual
```

答案很简单，换一种思考方式就可以简单解决问题。差点我就准备改程序把string类型改成float。一天的工作量一下就变成了一小时，又可以划水7小时了。

---

**参考资料**

- [解决Oracle出现以0开头的小数，开头的0消失的问题](https://blog.csdn.net/qq_34909807/article/details/76270987?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight)