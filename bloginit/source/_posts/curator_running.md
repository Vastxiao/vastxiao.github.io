---
title: Elastic Curator 的配置和运行
toc: true
categories:
  - elastic
tags:
  - elastic
  - curator
  - 配置
date: 2017-03-26 19:17:02
updated: 2017-03-26 19:17:02
---

# Elastic Curator 的配置和运行

elasticsearch索引的管理工具curator4的配置和运行详解,这个是elastic官方提供的工具.

## Singleton Command Line Interface

运行：

```sh
curator_cli [OPTIONS] COMMAND [ARGS]...
# 如果用不指定的Options连接es，可以指定curator.yml配置文件读取连接es和curator的参数
# 默认配置文件~/.curator/curator.yml
curator_cli --config /PATH/TO/curator.yml COMMAND [ARGS]...

# 帮助命令
curator_cli --help
curator_cli show_indices --help

# 例子
curator_cli --host 192.168.1.36:9200 show_indices
```

新版的curator_cli命令行工具对条件的匹配已经改了，
需要使用--filter_list JSON 的格式来过滤。(感觉已经没帮助可查了,用起来变得复杂- -！)

```sh
# 例如查看索引，需要使用--filter_list来过滤需要的索引
curator_cli --host 192.168.13.36:9200 show_indices  --filter_list '[{"filtertype":"age","source":"creation_date","direction":"older","unit":"days","unit_count":13},{"filtertype":"pattern","kind":"prefix","value":"logstash"}]'
```

这样还不如就用这个了：
curator [--config CONFIG.YML] [--dry-run] ACTION_FILE.YML

## Command Line Interface

运行：

```sh
curator [--config CONFIG.YML] [--dry-run] ACTION_FILE.YML

  --config CONFIG.YML    curator配置文件，配置连接es等参数，
                         默认为~/.curator/curator.yml
  --dry-run              测试运行
  ACTION_FILE.YML        YAML actionfile指定执行操作的文件
```

### ACTION_FILE.YML

ACTION_FILE.YML文件用于配置curator需要执行的操作。

https://www.elastic.co/guide/en/elasticsearch/client/curator/current/actionfile.html

#### curator_actions_cron.yml

主要用于配置curator定时执行任务，
将运行命令放在crontab中在每日凌晨0点过后执行，
用于清理和优化es的索引。

```yml
---
actions:
  1:
    action: delete_indices
    description: "删除过期索引"
    options:
      # action开关，为True表示不执行这个action，默认False
      disable_action: False
      # 如果为True，则在filters空列表时，继续下一个action处理而不是退出程序。
      ignore_empty_list: True
      # 发现错误后，继续执行下一个索引操作，默认False
      continue_if_exception: True
    filters:
    # 是否排除隐藏索引，如.kibana
    - filtertype: kibana
      exclude: True
    # 是否排除open状态的索引
    - filtertype: opened
      exclude: True
    # 处理30天前的索引
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 30
      exclude: False
  2:
    action: allocation
    description: "冷热数据分离任务：将非当天索引从hot节点迁移到stale节点。"
    options:
      # action开关，为True表示不执行这个action，默认False
      disable_action: False
      # ignore_empty_list标识忽略选项，默认False，
      # 如果为True，则在filters空列表时，继续下一个action处理而不是退出程序。
      ignore_empty_list: True
      # 发现错误后，继续执行下一个索引操作，默认False
      continue_if_exception: True
      allocation_type: include
      key: zone
      value: stale
      wait_for_completion: False
    filters:
    # 是否排除隐藏索引，如.kibana
    - filtertype: kibana
      exclude: True
    # 是否排除open状态的索引
    - filtertype: opened
      exclude: False
    # 处理1天前的索引
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 1
      exclude: False
  3:
    action: forcemerge
    description: "强制归并索引segments，优化es性能"
    options:
      # action开关，为True表示不执行这个action，默认False
      disable_action: False
      # ignore_empty_list标识忽略选项，默认False，
      # 如果为True，则在filters空列表时，继续下一个action处理而不是退出程序。
      ignore_empty_list: True
      # 发现错误后，继续执行下一个索引操作，默认False
      continue_if_exception: True
      max_num_segments: 1
      delay:
    filters:
    # 是否排除隐藏索引，如.kibana
    - filtertype: kibana
      exclude: False
    # 是否排除open状态的索引
    - filtertype: opened
      exclude: False
    # 处理1天前的索引
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 2
      exclude: False
  4:
    action: close
    description: "关闭匹配的索引，减少es内存占用"
    options:
      # action开关，为True表示不执行这个action，默认False
      disable_action: False
      # 如果为True，则在filters空列表时，继续下一个action处理而不是退出程序。
      ignore_empty_list: True
      # 发现错误后，继续执行下一个索引操作，默认False
      continue_if_exception: True
      # close索引前是否删除aliases
      delete_aliases: False
    filters:
    # 是否排除隐藏索引，如.kibana
    - filtertype: kibana
      exclude: True
    # 是否排除open状态的索引
    - filtertype: opened
      exclude: False
    # 处理7天前的索引
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 7
      exclude: False
```

#### curator_actions_cmd.yml

主要用于需要手动处理的es索引任务。

```yaml
---
actions:
  1:
    action: open
    description: "open匹配的索引"
    options:
      # action开关，为True表示不执行这个action，默认False 【记得手动开启action】
      disable_action: True
      # 如果为True，则在filters空列表时，继续下一个action处理而不是退出程序。
      ignore_empty_list: True
      # 发现错误后，继续执行下一个索引操作，默认False
      continue_if_exception: True
    # 下面开始匹配需要执行的任务的索引 【记得手动修改filters条件】
    filters:
    - filtertype: closed
      exclude: False
    # 是否排除open状态的索引
    - filtertype: opened
      exclude: True
    # 是否排除隐藏索引，如.kibana
    - filtertype: kibana
      exclude: True
    # 主要在这里匹配索引的时间
    - filtertype: age
      source: name
      # direction值为: younger|older
      direction: younger
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 14
      exclude: False
    # 这里匹配索引名称
    - filtertype: pattern
      # kind值: prefix|suffix|timestring|regex
      kind: prefix
      value: logstash-
      exclude: False
```

## Environment Variables

环境变量可以应用与执行curator和curator_cli命令。

环境变量支持写入CONFIG.YML和ACTION_FILE.YML。

https://www.elastic.co/guide/en/elasticsearch/client/curator/current/envvars.html

## CONFIG.YML

https://www.elastic.co/guide/en/elasticsearch/client/curator/current/configfile.html

CONFIG.YML是curator和curator_cli的全局配置文件。

在使用命令时，如果不指定配置文件，默认读取文件：

```
~/.curator/curator.yml
```

curator.yml

```yml
---
# Remember, leave a key empty if there is no value.  None will be a string,
# not a Python "NoneType"
client:
  hosts:
    - 10.0.0.1
    - 10.0.0.2:9200
    - 10.0.0.3:9201
  port: 9200
  url_prefix:
  use_ssl: False
  certificate:
  client_cert:
  client_key:
  ssl_no_validate: False
  http_auth:
  timeout: 30
  master_only: False

logging:
  loglevel: INFO
  logfile:
  logformat: default
  blacklist: ['elasticsearch', 'urllib3']
```

## curator4版本当前的坑说明

curator当前在filters中的age类型中对时间存在时差问题.

原因是,这个age对时间的计算形式如下:

```
point_of_reference = epoch - ((number of seconds in unit) * unit_count)

<point_of_reference>就是计算出来的时间
<epoch>是当前时间的utc时间
<number of seconds in unit>就是配置的day或hours单位的秒数,就是配置的unit换算来的.
<unit_count>就是unit_count值

curator直接是把<point_of_reference>这个时间拿来匹配索引的时间,比如索引名称的后缀时间%Y.%m.%d

由这里很明显的知道了由于<epoch>是utc时间,所以<point_of_reference>也utc时间.
那么,中国的和utc时间是要+8小时的,所以这里匹配出来的时间要再加8小时间才是对的.

当前看似官方也知道这个问题,后面的版本可能会再换算一个native时区的时间吧.

当前可以这样解决:
在换算时间的时候,有意识的先扣掉8个小时再换算,这样就可以解决问题了:

比如要获取前一天是索引,改成这样:
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: hours
      unit_count: 16
      exclude: False
```



