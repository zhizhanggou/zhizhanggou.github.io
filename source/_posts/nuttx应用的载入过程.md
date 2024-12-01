---
title: nuttx应用的载入过程
date: 2023-11-05 14:21:16
categories: 嵌入式
tags:
---
# nuttx应用的载入过程（cmake版）
以apps/examples/helloxx这个应用为例
其应用入口为apps/examples/helloxx/helloxx_main.cxx中的
`extern "C" int main(int argc, FAR char *argv[])`函数  
在该目录下CMakeLists.txt中将其添加到nsh的命令中   
```if(CONFIG_EXAMPLES_HELLOXX)  
nuttx_add_application(   
    NAME
    helloxx
    STACKSIZE
    ${CONFIG_DEFAULT_TASK_STACKSIZE}
    MODULE
    ${CONFIG_EXAMPLES_HELLOXX}
    SRCS
    helloxx_main.cxx)
endif()
```
在nuttx/cmake/nuttx_add_application.cmake中将创建一个名为apps_helloxx的library

```
set(TARGET "apps_${NAME}")
add_library(${TARGET} ${SRCS})
```
再通过nuttx_add_library_internal将nuttx相关的头文件以及依赖传进来。

修改helloxx_main.cxx中main函数的名字改为helloxx_main
```
list(GET SRCS 0 MAIN_SRC)
set_property(
    SOURCE ${MAIN_SRC}
    APPEND
    PROPERTY COMPILE_DEFINITIONS main=${NAME}_main)
```

将应用的主函数名字、应用名字、PRIORITY、STACKSIZE参数则传入对应的properties中


```
  set_target_properties(${TARGET} PROPERTIES APP_MAIN ${NAME}_main)
  set_target_properties(${TARGET} PROPERTIES APP_NAME ${NAME})

  if(PRIORITY)
    set_target_properties(${TARGET} PROPERTIES APP_PRIORITY ${PRIORITY})
  else()
    set_target_properties(${TARGET} PROPERTIES APP_PRIORITY
                                               SCHED_PRIORITY_DEFAULT)
  endif()

  if(STACKSIZE)
    set_target_properties(${TARGET} PROPERTIES APP_STACK ${STACKSIZE})
  else()
    set_target_properties(${TARGET} PROPERTIES APP_STACK
                                               ${CONFIG_DEFAULT_TASK_STACKSIZE})
```

