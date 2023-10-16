---
title: Linux下删除过期天数文件
author: starifly
date: 2019-10-14T17:23:19+08:00
categories: [linux]
tags: [linux]
draft: true
slug: delete-expired-days-files-under-linux
---

删除指定目录及其子目录过期文件：一种方法可以直接用find命令，另外一种方法是用递归函数结合 stat 查出文件的日期和大小来进行删除。

### find

```shell
#!/bin/bash
#--------------------------------------------
# useage:脚本名称 $1 $2 $3
# $1，第一个参数代表天数，超过即清除日志
# $2,$3，第二个参数代表天数，第三个参数代表大小，单位为M，此两个参数联合使用,同时成立才清除日志
#--------------------------------------------

log_path=/usr/local/tomcat/logs

function clean_log()
{
    days1=$1
    days2=$2
    size=$3
    pathdir=$4

    find $pathdir -regex '^.*\(log.*\.txt\|\.log\|catalina.*\.out\)$' -and -mtime +$days1 -type f -exec rm -rf {} \;
    find $pathdir -regex '^.*\(log.*\.txt\|\.log\|catalina.*\.out\)$' -and -mtime +$days2 -size +${size}M -type f -exec rm -rf {} \;

    return 0;
}

root_dir="."
function main()
{
    if [ $# -eq 3 ]
    then
        cd $log_path
        if [ $? -eq 0 ]
        then
            clean_log $1 $2 $3 $root_dir
            ret=$?
            echo "read_dir run ret:$ret"
        fi
    fi
}

main $@
```

### 遍历方法

```shell
#!/bin/bash
#--------------------------------------------
# useage:脚本名称 $1 $2 $3
# $1，第一个参数代表天数，超过即清除日志
# $2,$3，第二个参数代表天数，第三个参数代表大小，单位为M，此两个参数联合使用,同时成立才清除日志
#--------------------------------------------

log_path=/usr/local/tomcat/logs

function clean_log()
{
    INTERVAL=$(($1*3600*24))
    INTERVAL2=$(($2*3600*24))
    limit_size=$(($3*1024*1024))
    pathdir=$4
    now_timestamp=$(date -d "$(date +"%Y-%m-%d %T")" +%s)
    files=$(ls $pathdir/*.txt $pathdir/*.log $pathdir/*.out -1)

    for file in $files;
    do
        if [ -f "$file" ]
        then
            file_date=$( stat $file | tail -3|head -1 | awk '{print $2,$3}' )
            file_size=$( stat -c "%s" "$file" )
            file_timestamp=$(date -d "$file_date" +%s)

            if [ $? -ne 0 ]
            then
                file_path=$file
                echo "delete file 0 $file_path"
                #rm -rf $file_path
                continue
            fi

            if [ $(($now_timestamp - $file_timestamp)) -gt $INTERVAL ]
            then
                file_path=$file
                echo "delete file 1 $file_path logger than $1 days"
                rm -rf $file_path
            elif [ $(($now_timestamp - $file_timestamp)) -gt $INTERVAL2 ] && [ $file_size -gt $limit_size ]
            then
                file_path=$file
                echo "delete file 1 $file_path logger than $2 days and bigger than $3 M"
                rm -rf $file_path
            fi
        fi
    done
    return 0;
}

function read_dir()
{
    for element in `ls $4`
    do
        dir_or_file=$4"/"$element
        if [ -d $dir_or_file ]
        then
            clean_log $1 $2 $3  $dir_or_file
            read_dir $1 $2 $3 $dir_or_file
        fi
    done
    return 0;
}

root_dir="."
function main()
{
    if [ $# -eq 3 ]
    then
        cd $log_path
        if [ $? -eq 0 ]
        then
            clean_log $1 $2 $3 $root_dir
            read_dir $1 $2 $3 $root_dir
            ret=$?
            echo "read_dir run ret:$ret"
        fi
    fi
}

main $@
```
