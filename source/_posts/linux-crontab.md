---
title: linux_crontab
date: 2019-07-31 14:33:24
tags: [linux, crontab]
---

## 创建crontab任务
　　利用 *crontab -e* 可以调用vi以交互的方式直接创建提交crontab任务，这里由于需求，主要记录一下使用脚本直接创建crontab任务的方法，参考了[blog](https://www.jianshu.com/p/5ffb0fbe5fe9)、[输出重定向](https://unix.stackexchange.com/questions/64736/combine-output-from-two-commands-in-bash)。  
<!--more-->
　　使用方式，主要是利用 *crontab -l* 的输出，并对输出做一些处理后作为 *crontab* 的输入，从而完成创建和删除crontab任务。  

```shell
TASK_NAME="myCrontabJob.py"
cd $(dirname "$0")
WORK_DIR="$(pwd)"
TASK_COMMAND="python ${WORK_DIR}/${TASK_NAME}"
CRONTAB_TASK="*/1 * * * * ${TASK_COMMAND} >/dev/null 2>&1"

function create_crontab()
{
    ( crontab -l 2>/dev/null | grep -v "${TASK_NAME}";echo "${CRONTAB_TASK}" ) | crontab -
}

function remove_crontab()
{
    crontab -l | grep -v "${TASK_NAME}" | crontab -
}

if [ $1 = 'create' ];then
    create_crontab
elif [ $1 = 'remove' ];then
    remove_crontab
else
    echo "arguments : create/remove"
    exit 1
fi
```

　　主要分析一下：  

```
( crontab -l 2>/dev/null | grep -v "${TASK_NAME}";echo "${CRONTAB_TASK}" ) | crontab -  
```

　　*crontab -l* 主要是输出当前用户的crontab任务  
　　*grep -v "${TASK_NAME}"* 是如果输出中包括当前要创建的任务，则过滤，即删掉避免重复  
　　*echo "${TASK_NAME}"* 则是输出要创建的任务，相当于追加到 *crontab -l* 的输出上  
　　*crontab -* 将前面的输出结果作为未命名文件并作为输入，从而更新crontab任务表，从而实现任务添加  
　　主要利用了多条命令的io重定向，上面国外论坛参考链接中有解释:  

```shell
( command1 ; command2 ; command3 ) | cat
```
