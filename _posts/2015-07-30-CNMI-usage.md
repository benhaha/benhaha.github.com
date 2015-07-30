---
layout: post
title: how to use CNMI
date: 2015-07-30 16:27:31
---

常见缩写含义：

- ME (Mobile Equipment)
- MS (Mobile Station)
- TA (Terminal Adapter)
- MT (Mobile Termial)
- DCE (Data Communication Equipment)
- DTE (Data Terminal Equipment)

CNMI命令解析

CNMI(New Message Indication)命令用来设置在接收到新短消息时的处理方法。

命令格式为：

AT+CNMI=[<mode>[,<mt>[,<bm>[,<ds>[,<bfr>]]]]]

各参数含义如下

mode：

- 0 在MT中缓存URCs，存满后循环覆盖
- 1 在MT-DTE之间连接留作它用时，忽略指示及新接收消息URCs，否则直接将消息发送到DTE
- 2 当串口忙时（例如在传输数据），在MT中缓存URCs，否则直接将消息发送到DTE

*-note-*

**0意味着MT不会通知短信接收；1，2会提示，但2会缓存，1会直接抛弃**

mt：

-0 无SMS_DELIVER指示发送到TE。
-1 如果SMS_DELIVER存储在MT，会使用+CMTI URC向DTE指示内存位置。
-2 SMS_DELIVER直接发送到DTE（不会存储到模块或SIM卡中），使用+CMT URC。

*-note-*

**实际使用AT+CNMI=2,2在处理时比较方便，相当于直接设置将短信发送给DTE**

编程实现：

首先解析CNMI后边字符串，得到字符串信息后，再次进行自定义命令匹配，实现设备重启机制。
