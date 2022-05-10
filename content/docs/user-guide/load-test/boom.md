---
title: hrp boom
weight: 1
description: 通过 hrp boom 命令行工具启动性能测试
---

`HttpRunner` supports running load test without extra work. You can use `hrp boom` command to run YAML/JSON testcases in load testing mode.

By default, hrp will print load testing results in console output, refreshed every 3 seconds.

```
$ hrp boom examples/demo.json --spawn-count 10 --spawn-rate 1
6:09PM INF Set log to pretty console
6:09PM INF Set log level to INFO
6:09PM INF Set log level to WARN
2021/12/02 18:09:48 Spawning 10 clients immediately
Current time: 2021/12/02 18:09:51, Users: 10, Total RPS: 20, Total Fail Ratio: 0.0%
+--------------+-----------------+------------+---------+--------+---------+------+------+--------------+------------+-------------+
|     TYPE     |      NAME       | # REQUESTS | # FAILS | MEDIAN | AVERAGE | MIN  | MAX  | CONTENT SIZE | # REQS/SEC | # FAILS/SEC |
+--------------+-----------------+------------+---------+--------+---------+------+------+--------------+------------+-------------+
| request-GET  | get with params |         10 |       0 |   2400 | 2423.00 | 2422 | 2424 |          300 |         10 |           0 |
| request-POST | post json data  |         10 |       0 |    310 |  304.50 |  301 |  307 |          420 |         10 |           0 |
+--------------+-----------------+------------+---------+--------+---------+------+------+--------------+------------+-------------+

Current time: 2021/12/02 18:09:54, Users: 10, Total RPS: 16, Total Fail Ratio: 0.0%
+--------------+-----------------+------------+---------+--------+---------+------+------+--------------+------------+-------------+
|     TYPE     |      NAME       | # REQUESTS | # FAILS | MEDIAN | AVERAGE | MIN  | MAX  | CONTENT SIZE | # REQS/SEC | # FAILS/SEC |
+--------------+-----------------+------------+---------+--------+---------+------+------+--------------+------------+-------------+
| request-GET  | get with params |         18 |       0 |   1200 | 1157.39 | 1083 | 1367 |          300 |          9 |           0 |
| request-POST | post json data  |         10 |       0 |    290 |  290.20 |  287 |  293 |          420 |         10 |           0 |
| request-POST | post form data  |         20 |       0 |    310 |  300.00 |  287 |  311 |          441 |         10 |           0 |
+--------------+-----------------+------------+---------+--------+---------+------+------+--------------+------------+-------------+

Current time: 2021/12/02 18:09:57, Users: 10, Total RPS: 17, Total Fail Ratio: 0.0%
+--------------+-----------------+------------+---------+--------+---------+------+------+--------------+------------+-------------+
|     TYPE     |      NAME       | # REQUESTS | # FAILS | MEDIAN | AVERAGE | MIN  | MAX  | CONTENT SIZE | # REQS/SEC | # FAILS/SEC |
+--------------+-----------------+------------+---------+--------+---------+------+------+--------------+------------+-------------+
| request-GET  | get with params |         12 |       0 |   1100 | 1153.92 | 1081 | 1464 |          300 |          6 |           0 |
| request-POST | post json data  |         20 |       0 |    270 |  279.70 |  269 |  337 |          420 |          6 |           0 |
| request-POST | post form data  |         20 |       0 |    270 |  272.85 |  269 |  279 |          441 |         10 |           0 |
+--------------+-----------------+------------+---------+--------+---------+------+------+--------------+------------+-------------+
```

If you want to disable console output, you can add a `--disable-console-output` flag.

```
$ hrp boom examples/demo.json --spawn-count 10 --spawn-rate 1 --disable-console-output
```

You can reference this [doc][hrp_boom] for all command arguments.

[hrp_boom]: https://github.com/httprunner/httprunner/blob/master/docs/cmd/hrp_boom.md
