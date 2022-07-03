---
title: C++提取字符串中的数字(strtok_s的使用)
date: 2019-07-02 21:50:15
categories: C++日常
tags: 字符串
---


​		之前遇到需要将表示时间的字符串，例如“1995-10-08”，将其中的年月日提取出来用int型存储。C++(11)下没有直接转换标准时间格式的函数。直接用if语句提取来进行判断并截取十分的麻烦费时。

​		可以用**strtok_s**(linux下为**strtok_r**)函数进行处理,其形式为:

```c++
char *strtok_s( char *strToken, const char *strDelimit, char **buf);


```

​		其返回值是被切割的第一个子字符串。strToken为被切割的字符串，strDelimit包含分隔字符的字符串，buf为切割后剩余字符串。**字符串切割完后会将原字符串中的分隔字符替换为\0**。

使用方法如下：

```c++
typedef struct Date
{
	int year;
	int month;
	int day;
};

void DateStrToInt(string date_S, Date &date_I)
{
	string temp;
	char* buf;
	temp = strtok_s((char*)date_S.data(), "-", &buf);
	date_I.year = stoi(temp);
	temp = strtok_s(NULL, "-", &buf);
	date_I.month = stoi(temp);
	date_I.day = stoi(buf);
}
```

需注意的是strDelimit为字符串的话并不是以整个字符串作为分隔符，而是其中每个字符单独作为分隔符。

针对在线笔试的话直接用函数

```
scanf("%d-%d-%d",&year,&month,&day);
```

就完事了，不用麻烦的去操作字符串，太浪费时间。

