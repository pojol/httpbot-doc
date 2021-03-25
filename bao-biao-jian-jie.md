# 报表简介



> httpbot 的repot 分成了三个部分（如下

```shell
$1
/v1/login/guest           Req count 1     Consume 21ms  Succ rate 1/1   0kb / 0kb

$2
+----------------------------------------------------------------------------+
Req url                   Req count       Average time       Succ rate
/v1/login/guest           1               21ms               1/1        0kb / 0kb
+-----------------------------------------------------------------------------+
robot : 1 req count : 1 duration : 1s qps : 1 errors : 0

$3
http://123.207.198.57:14001/v1/login/guest                   match 0
coverage 0 / 1
```



* 第一部分（单个请求的报告

  ```shell
  # url path								请求次数				 请求消耗时间（毫秒 成功率        请求/回复 包体大小
  /v1/login/guest           Req count 1     Consume 21ms  Succ rate 1/1   0kb / 0kb
  ```

  

* 第二部分（统计报告

  ```shell
  # 上边框体内的报告是，整个工厂周期内所产生的请求报告
  +----------------------------------------------------------------------------+
  Req url                   Req count       Average time       Succ rate
  /v1/login/guest           1               21ms               1/1        0kb / 0kb
  +-----------------------------------------------------------------------------+
  # 一共创建了多少个bot   总的请求数量     工厂的生产持续时间   QPS    错误数
  robot : 1             req count : 1  duration : 1s    qps : 1 errors : 0
  ```

  

* 第三部分（测试覆盖率报告

  ```shell
  # 请求的调用次数
  http://123.207.198.57:14001/v1/login/guest                   match 0
  coverage 0 / 1		# 请求的覆盖率
  ```

  