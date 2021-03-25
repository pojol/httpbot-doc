# Benchmark



> 腾讯云 4核 QPS大致在2.5w+，压测代码在 factory/factory_test.go

```text
[root@VM_0_3_centos factory]# go test -bench=.
goos: linux
goarch: amd64
pkg: github.com/pojol/httpbot/factory
BenchmarkFactoryStatic-4   	      30	  40094525 ns/op
PASS
+--------------------------------------------------------------------------------+
Req url                       Req count       Average time       Succ rate
                              25997           1ms                45997/45997 0kb / 179kb
+--------------------------------------------------------------------------------+
robot : 25997 req count : 25997 duration : 1s qps : 25997 errors : 0
```

