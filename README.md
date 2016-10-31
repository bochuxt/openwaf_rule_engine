Name
====

openwaf_rule_engine是[OPENWAF](https://github.com/titansec/openwaf)的规则引擎

Table of Contents
=================

* [Name](#name)
* [Version](#version)
* [Synopsis](#synopsis)
* [Description](#description)
* [Community](#community)
* [Bugs and Patches](#bugs-and-patches)
* [Changes](#changes)
* [Modules Configuration Directives](#modules-configuration-directives)
* [Rule Directives](#rule-directives)
* [Variables](#variables)
* [Transformation Functions](#transformation-functions)
* [Operators](#operators)
* [Options](#options)

Version
=======

This document describes OpenWAF Rule Engine v0.0.1.161026_beta released on 26 Oct 2016.

Synopsis
========


[Back to TOC](#table-of-contents)

Description
===========

规则引擎的启发来自[modsecurity](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual)及[freewaf(lua-resty-waf)](https://github.com/p0pr0ck5/lua-resty-waf)，将ModSecurity的规则机制用lua实现。基于规则引擎可以进行协议规范，自动工具，注入攻击，跨站攻击，信息泄露，异常请求等安全防护，支持动态添加规则，及时修补漏洞。

[Back to TOC](#table-of-contents)

Community
=========

English Mailing List
--------------------

The [OpenWAF-en](https://groups.google.com/group/openwaf-en) mailing list is for English speakers.

Chinese Mailing List
--------------------

The [OpenWAF-cn](https://groups.google.com/group/openwaf-cn) mailing list is for Chinese speakers.

Personal QQ Mail
----------------

290557551@qq.com

[Back to TOC](#table-of-contents)

Bugs and Patches
================

Please submit bug reports, wishlists, or patches by

1. creating a ticket on the [GitHub Issue Tracker](https://github.com/titansec/OpenWAF/issues),
1. or posting to the [TWAF community](#community).

[Back to TOC](#table-of-contents)

Changes
=======

[Back to TOC](#table-of-contents)


Modules Configuration Directives
================================
```txt
    "twaf_secrules":{
        "state": true,                                              -- 总开关
        "reqbody_state": true,                                      -- 请求体检测开关
        "header_filter_state": true,                                -- 响应头检测开关
        "body_filter_state": true,                                  -- 响应体检测开关
        "reqbody_limit":134217728,                                  -- 请求体检测阈值，大于阈值不检测
        "respbody_limit":524288,                                    -- 响应体检测阈值，大于阈值不检测
        "pre_path": "/opt/OpenWAF/",                                -- OpenWAF安装路径
        "path": "lib/twaf/inc/knowledge_db/twrules",                -- 特征规则库在OpenWAF中的路径
        "msg": [                                                    -- 日志格式
            "category",
            "severity",
            "action",
            "meta",
            "version",
            "id",
            "charactor_name",
            {                                                       -- 字典中为变量
                "transaction_time": "%{DURATION}",
                "logdata": "%{MATCHED_VAR}"
            }
        ],
        "user_defined_rules":[                                      -- 用户自定义规则，数组
        ],
        "rules_id":{                                                -- 特征排除
            "111112": [{"REMOTE_HOST":"a.com", "URI":"^/ab"}]       -- 匹配中数组中信息则对应规则失效，数组中key为变量名称，值支持正则
            "111113": []                                            -- 特征未被排除
            "111114": [{}]                                          -- 特征被无条件排除
        }
    }
```

###state
**syntax:** *"state": true|false*

**default:** *true*

**context:** *twaf_secrules*

规则引擎总开关

###reqbody_state
**syntax:** *"reqbody_state"" true|false*

**default:** *true*

**context:** *twaf_secrules*

请求体检测开关

###header_filter_state
**syntax:** *"header_filter_state": true|false*

**default:** *true*

**context:** *twaf_secrules*

响应头检测开关

###body_filter_state
**syntax:** *"body_filter_state": true|false*

**default:** *false*

**context:** *twaf_secrules*

响应体检测开关，默认关闭，若开启需添加第三方模块[ngx_http_twaf_header_sent_filter_module暂未开源]

###reqbody_limit
**syntax:** *"reqbody_limit": number*

**default:** *134217728*

**context:** *twaf_secrules*

请求体检测大小上限，默认134217728B(128MB)，若请求体超过设置上限，则不检测

PS：reqbody_limit值要小于nginx中client_body_buffer_size的值才会生效

###respbody_limit
**syntax:** *"respbody_limit": number*

**default:** *134217728*

**context:** *twaf_secrules*

响应体检测大小上限，默认134217728B(128MB)，若响应体大小超过设置上限，则不检测

###pre_path
**syntax:** *"pre_path" string*

**default:** */opt/OpenWAF/*

**context:** *twaf_secrules*

OpenWAF的安装路径

###path
**syntax:** *"path": string*

**default:** *lib/twaf/inc/knowledge_db/twrules*

**context:** *twaf_secrules*

特征规则库在OpenWAF中的路径

###msg
**syntax:** *"msg": table*

**default:** *[
            "category",
            "severity",
            "action",
            "meta",
            "version",
            "id",
            "charactor_name",
            {
                "transaction_time": "%{DURATION}",
                "logdata": "%{MATCHED_VAR}"
            }
        ]*

**context:** *twaf_secrules*

日志格式

###user_defined_rules
**syntax:** *"user_defined_rules": table*

**default:** *none*

**context:** *twaf_secrules*

策略下的用户自定义特征规则

系统特征规则适用于所有的策略，在引擎启动时通过加载特征库或通过API加载系统特征规则，系统特征规则一般不会动态更改

用户自定义特征在策略下生效，一般用于变动较大的特征规则，如：时域控制，修改响应头等临时性规则

```json
"user_defined_rules":[
    {
        "weight": 0,
        "id": "1000001",
        "release_version": "858",
        "charactor_version": "001",
        "disable": false,
        "opts": {
            "nolog": false,
            "sanitise_arg": [
                "password",
                "passwd"
            ]
        },
        "phase": "access",
        "action": "deny",
        "meta": 403,
        "severity": "high",
        "category": "user defined rule-time",
        "charactor_name": "relative time",
        "desc": "周一至周五的8点至18点，禁止访问/test目录",
        "match": [{
            "vars": [{
                "var": "URI"
            }],
            "operator": "begins_with",
            "pattern": "/test"
        },
        {
            "vars": [{
                "var": "TIME_WDAY"
            }],
            "operator": "equal",
            "pattern": ["1", "2", "3", "4", "5"]
        },
        {
            "vars": [{
                "var": "TIME"
            }],
            "operator": "str_range",
            "pattern": ["08:00:00-18:00:00"]
        }]
    },
    {
        "weight": 0,
        "id": "1000002",
        "release_version": "858",
        "charactor_version": "001",
        "disable": false,
        "opts": {
            "nolog": false,
            "sanitise_arg": [
                "password",
                "passwd"
            ]
        },
        "phase": "access",
        "action": "deny",
        "meta": 403,
        "severity": "high",
        "category": "user defined rule-iputil",
        "charactor_name": "iputil",
        "desc": "某ip段内不许访问",
        "match": [{
            "vars": [{
               "var": "REMOTE_ADDR"
            }],
            "operator": "ip_utils",
            "pattern": ["1.1.1.0/24","2.2.2.2-2.2.20.2"]
        }]
    }
]
```

###rules_id
**syntax:** *rules_id table*

**default:** *none*

**context:** *twaf_secrules*

用于排除特征

[Back to TOC](#table-of-contents)

Rule Directives
===============
```txt
-- lua格式
    {
        id = "000001",                             -- ID标识(唯一)，string类型
        release_version = "858",                   -- 特征库版本，string类型
        charactor_version = "001",                 -- 特征规则版本，string类型
        severity = "low",                          -- 严重等级，OPENWAF中使用"low"，"medium","high"等，string类型
        category = "test",                         -- 分类，string类型
        charactor_name = "test",                   -- 特征名称，string类型
        opts = {                                   -- 其余动作
            nolog = false,                         -- 不记日志，true or false，默认false
            add_resp_headers = {                   -- 自定义响应头
                key_xxx = "value_xxx"              -- 响应头名称 = 响应头值
            }
            setvar = {{                            -- 设置变量，数组
                column = "test",                   -- 变量的一级key，如：TX，session等，string类型
                key = "test",                      -- 变量的二级key，如：score, id等，string类型
                incr = true,                       -- 同modsec中的=+操作， true or false，默认false
                value = 5,                         -- 变量值，number类型
                time = 3000                        -- 超时时间(ms)，number类型
            }}
        },
        phase = "test",                            -- 执行阶段（"access","header_filter","body_filter"），支持数组和字符串
        action = "test",                           -- 动作，ALLOW，DENY等，string类型
        desc = "test",                             -- 描述语句
        tags = {"test1", "test2"},                 -- 标签
        match = {                                  -- match之间是与的关系，数组
            {
                vars = {{                          -- 数组取代modsec中的"|"，处理多个var
                    var = "test",                  -- 变量名称，string类型
                    phase = "test",                -- 执行阶段，若为空，则查找上一级别的phase字段，string类型
                    parse = {                      -- 对变量的解析，下面的操作只能出现一种
                        specific = "test",         -- 具体化，取代modsec中的":",支持数组，TODO:支持正则，如modsec的"/ /"，支持字符串和数组
                        ignore = "test",           -- 忽略某个值,取代modsec中的"!"，TODO:支持正则，如modsec的"/ /"，支持字符串和数组
                        keys = true,               -- 取出所有的key，true or false，默认false
                        values = "test",           -- 取出所有的value，true or false，默认false
                        all = "test"               -- 取出所有的key和value，true or false，默认false
                    }
                }},
                transform = "test",                -- 转换操作,支持字符串和数组
                operator = "test",                 -- 操作，string类型
                pattern = "test",                  -- 操作参数，支持boolean、number、字符串和数组
                pf = "file_path",                  -- 操作参数，文件路径
                op_negated = true                  -- 操作取反，true or false，默认false
            },
            {match_info2},
            {match_info3},
            ....
        }
    }

--json格式
    {
        "id": "000001",
        "release_version": "858",
        "charactor_version": "001",
        "severity": "test",
        "category": "test",
        "charactor_name": "test",
        "opts": {
            "nolog": false,
            "add_resp_headers": {
                "key_xxx": "value_xxx"
            },
            "setvar": [{
                "column":"test",
                "key":"test",
                "incr": true,
                "value": 5,
                "time":3000
            }]
        },
        "phase": "test",
        "action": "test",
        "desc": "test",
        "tags": ["test1", "test2"]
        "match": [
            {
                "vars": [{
                    "var": "test",
                    "storage": true,
                    "phase": "test",
                    "parse": {
                        "specific": "test",
                        "ignore": "test",
                        "keys": true,
                        "values": "test",
                        "all": "test"
                    }
                }],
                "transform": "test",
                "operator": "test",
                "pattern": "test",
                "pf": "file_path",
                "op_negated": true
            },
            {"match_info2"},
            {"match_info3"},
            ...
        ]
    }
```

Variables
=========
* [ARGS](#args)
* [ARGS_COMBINED_SIZE](#args_combined_size)
* [ARGS_GET](#args_get)
* [ARGS_GET_NAMES ](#args_get_names)
* [ARGS_NAMES](#args_names)
* [ARGS_POST ](#args_post)
* [ARGS_POST_NAMES ](#args_post_names)
* [DURATION](#duration)
* [FILES ](#files)
* [FILES_NAMES](#files_names)
* [GEO](#geo)
* [HTTP_HOST](#http_host)
* [MATCHED_VAR](#matched_var)
* [MATCHED_VARS](#matched_vars)
* [MATCHED_VAR_NAME](#matched_var_name)
* [MATCHED_VARS_NAMES](#matched_var_names)
* [QUERY_STRING](#query_string)
* [REMOTE_ADDR](#remote_addr)
* [REMOTE_HOST](#remote_host)
* [REMOTE_PORT](#remote_port)
* [REMOTE_USER](#remote_user)
* [REQUEST_BASENAME](#request_basename)
* [REQUEST_BODY](#request_body)
* [REQUEST_COOKIES](#request_cookies)
* [REQUEST_COOKIES_NAMES](#request_cookies_names)
* [REQUEST_FILENAME](#request_filename)
* [REQUEST_HEADERS](#request_headers)
* [REQUEST_HEADERS_NAMES](#request_headers_names)
* [REQUEST_LINE](#request_line)
* [REQUEST_METHOD](#request_method)
* [REQUEST_PROTOCOL](#request_protocol)
* [HTTP_VERSION](#http_version)
* [URI](#uri)
* [REQUEST_URI](#request_uri)
* [RESPONSE_BODY](#response_body)
* [RESPONSE_HEADERS](#response_headers)
* [RESPONSE_STATUS](#response_status)
* [SCHEME](#scheme)
* [SERVER_ADDR](#server_addr)
* [SERVER_NAME](#server_name)
* [SERVER_PORT](#server_port)
* [SESSION](#session)
* [SESSION_DATA](#session_data)
* [TIME](#time)
* [TIME_DAY](#time_day)
* [TIME_EPOCH](#time_epoch)
* [TIME_HOUR](#time_hour)
* [TIME_MIN](#time_min)
* [TIME_MON](#time_mon)
* [TIME_SEC](#time_sec)
* [TIME_WDAY](#time_wday)
* [TIME_YEAR](#time_year)
* [TX](#tx)
* [UNIQUE_ID](#unique_id)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS

table类型，所有的请求参数，包含ARGS_GET和ARGS_POST，但只检验value，不检验NAME

PS: 此变量含义同modsecurity的ARGS变量

```
例如：POST http://www.baidu.com?name=miracle&age=5

请求体为：time=123456&day=365

ARGS变量值为["miracle", "5", "123456", "365"]
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_COMBINED_SIZE

number类型，请求参数总长度，只包含key和value的长度，不包含'&'或'='等符号

PS: 此变量含义同modsecurity的ARGS_COMBINED_SIZE变量

```
例如：GET http://www.baidu.com?name=miracle&age=5

ARGS_COMBINED_SIZE变量值为15，而不是18
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_GET

table类型，querystring参数

PS: 此变量含义同modsecurity的ARGS_GET变量，同freewaf的URI_ARGS变量

```
例如：GET http://www.baidu.com?name=miracle&age=5

ARGS_GET变量值为["miracle", "5"]
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_GET_NAMES

table类型，querystring参数key值

PS: 此变量含义同modsecurity的ARGS_GET_NAMES变量

```
例如：GET http://www.baidu.com?name=miracle&age=5

ARGS_GET_NAMES变量值为["name", "age"]
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_NAMES

table类型，querystring参数key值及post参数key值

PS: 此变量含义同modsecurity的ARGS_NAMES变量

```
例如：POST http://www.baidu.com?name=miracle&age=5

请求体为：time=123456&day=365

ARGS_NAMES变量值为["name", "age", "time", "day"]
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_POST

table类型，POST参数

PS: 此变量含义同modsecurity的ARGS_POST变量

```
例如：

POST http://www.baidu.com/login.html

请求体为：time=123456&day=365

ARGS_POST变量值为["123456", "365"]
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_POST_NAMES

table类型，POST参数key值

PS: 此变量含义同modsecurity的ARGS_POST_NAMES变量

```
例如：

POST http://www.baidu.com/login.html

请求体为：time=123456&day=365

ARGS_POST_NAMES变量值为["time", "day"]
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##DURATION

string类型，处理事务用时时间

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##FILES

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##FILES_NAMES

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##GEO

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##HTTP_HOST

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##MATCHED_VAR

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##MATCHED_VARS

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##MATCHED_VAR_NAME

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##MATCHED_VARS_NAMES

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##QUERY_STRING

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REMOTE_ADDR

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REMOTE_HOST

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REMOTE_PORT

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REMOTE_USER

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_BASENAME

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_BODY

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_COOKIES

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_COOKIES_NAMES

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_FILENAME

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_HEADERS

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_HEADERS_NAMES

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_LINE

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_METHOD

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_PROTOCOL

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##HTTP_VERSION

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##URI

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_URI

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##RESPONSE_BODY

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##RESPONSE_HEADERS

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##RESPONSE_STATUS

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SCHEME

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SERVER_ADDR

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SERVER_NAME

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SERVER_PORT

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SESSION

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SESSION_DATA

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_DAY

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_EPOCH

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_HOUR

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_MIN

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_MON

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_SEC

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_WDAY

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_YEAR

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TX

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##UNIQUE_ID

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

Transformation Functions
========================
* [base64_decode](#base64_decode)
* [sql_hex_decode](#sql_hex_decode)
* [base64_encode](#base64_encode)
* [counter](#counter)
* [compress_whitespace ](#compress_whitespace )
* [hex_decode](#hex_decode)
* [hex_encode](#hex_encode)
* [html_decode](#html_decode)
* [length](#length)
* [lowercase](#lowercase)
* [md5](#md5)
* [normalise_path](#normalise_path)
* [remove_nulls](#remove_nulls)
* [remove_whitespace](#remove_whitespace)
* [replace_comments](#replace_comments)
* [remove_comments_char](#remove_comments_char)
* [remove_comments](#remove_comments)
* [uri_decode](#uri_decode)
* [uri_encode](#uri_encode)
* [sha1](#sha1)
* [trim_left](#trim_left)
* [trim_right](#trim_right)
* [trim](#trim)

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##base64_decode

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##sql_hex_decode

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##base64_encode

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##counter

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##compress_whitespace

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##hex_decode

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##hex_encode

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##html_decode

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##length

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##lowercase

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##md5

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##normalise_path

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##remove_nulls

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##remove_whitespace

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##replace_comments

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##remove_comments_char

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##remove_comments

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##uri_decode

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##uri_encode

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##sha1

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##trim_left

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##trim_right

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##trim

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

Operators
=========

* [begins_with](#begins_with)
* [contains](#contains)
* [contains_word](#contains_word)
* [detect_sqli](#detect_sqli)
* [detect_xss](#detect_xss)
* [ends_with](#ends_with)
* [equal](#equal)
* [greater_eq](#greater_eq)
* [greater](#greater)
* [ip_utils](#ip_utils)
* [less_eq](#less_eq)
* [less](#less)
* [pf](#pf)
* [regex](#regex)
* [str_match](#str_match)
* [validate_url_encoding](#validate_url_encoding)
* [num_range](#num_range)
* [str_range](#str_range)

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##begins_with

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##contains

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##contains_word

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##detect_sqli

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##detect_xss

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##ends_with

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##equal

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##greater_eq

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##greater

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##ip_utils

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##less_eq

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##less

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##pf

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##regex

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##str_match

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##validate_url_encoding

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##num_range

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##str_range

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)


Options
=======

[Back to OPTIONS](#options)

[Back to TOC](#table-of-contents)
