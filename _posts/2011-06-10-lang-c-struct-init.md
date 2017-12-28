---
layout:     post
title:      "结构体乱序初始化"
subtitle:   ""
date:       2011-06-10 12:00:00
author:     "lyremelody"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C/C++
---


今天看到如下代码，感觉比较诧异

    struct bus_type usb_bus_type = { 
        .name = "usb",
        .match = usb_device_match,
    };  

我就在想，C/C++的变量命名规则好像不能使用“.”，然后我查了下bus_type的定义，结果如下：

    struct bus_type {
        const char  *name;
        ... 
    };  

这说明可能是结构体的一种初始化方法。一查，果然是
这是结构体的C风格乱序初始化。还有一种C++风格的乱序初始化，如下：

    struct bus_type usb_bus_type = { 
        name:   "usb",
        match:  usb_device_match,
    };  

