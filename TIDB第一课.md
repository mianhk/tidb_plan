TiDB-第一课

# 学习记录

暂无

# 作业



题目描述：

本地下载 TiDB，TiKV，PD 源代码，改写源码并编译部署以下环 境：

● 1 TiDB

● 1 PD

● 3 TiKV

改写后

● 使得 TiDB 启动事务时，会打一个 “hello transaction” 的 日志



# 完成过程

> 由于之前使用tiup在Linux机器上安装过TIDB，但是还没看过代码，这次就先下载了一下代码

## 步骤1： 下载源码并进行编译

直接把代码clone下来，对于TIDB和PD，由于是golang写的，直接分别执行make，即可编译出二进制文件。

tikv使用rust，先本地安装rust环境，需要确定再执行make进行编译。

## 步骤2： TiUP进行本机部署

根据说明，采用TiUP命令采用本地的二进制文件本机部署，但是由于tikv未本地编译完成，该步骤后续增加。

```
tiup playground --db.binpath ./pingcap/tidb/bin/tidb-server --kv.binpath ./pingcap/tidb/bin/pd-server pd.binpath ./pingcap/tidb/bin/tikv-server  
```

后续改写验证还是采用TiDB二进制启动进行验证。

## 步骤3：改写代码

改写内容： 

```go
// file:session/session.go
// line 1048
func (s *session) Execute(ctx context.Context, sql string) (recordSets []sqlexec.RecordSet, err error) {
  // begin
	logutil.Logger(ctx).Warn(sql)
	if strings.ToLower(sql) == "begin"{
		logutil.Logger(ctx).Warn("hello transaction")
	}
	// end
	if span := opentracing.SpanFromContext(ctx); span != nil && span.Tracer() != nil {
		span1 := span.Tracer().StartSpan("session.Execute", opentracing.ChildOf(span.Context()))
		defer span1.Finish()
		ctx = opentracing.ContextWithSpan(ctx, span1)
		logutil.Eventf(ctx, "execute: %s", sql)
	}
```



实验结果：

```
[2020/08/16 21:36:07.896 +08:00] [WARN] [session.go:1049] [begin]
[2020/08/16 21:36:07.896 +08:00] [WARN] [session.go:1052] ["hello transaction"]
[2020/08/16 21:36:07.896 +08:00] [WARN] [session.go:1049] [commit]
```

### 过程

刚刚开始直接在前面匹配了`begin`，然后在log中打印`hello transaction`，发现每次执行SQL都会生成log，怀疑excute是包含了自动提交的语句(begin,commit)，于是提前打印了SQL，得到了预期的结果。



## 收获

这节课主要的内容主要还是了解TIDB的基础架构，最主要的三块内容：存储、计算、调度，以及之间的协同。



由于这周时间比较紧，没有对TIDB代码进行过多的进行研究，只是在作业方便稍微看了一下，希望之后的课程能提前合理安排好时间，多花点时间深入的思考和研究。另外还有未完善的地方需要之后进行补充。









> 参考内容：
>
> 1. TiUP单机部署: [本地快速部署 TiDB 集群](https://docs.pingcap.com/zh/tidb/dev/tiup-playground)
> 2. TiDB SQL解析相关：[TiDB 源码阅读系列文章（三）SQL 的一生](https://pingcap.com/blog-cn/tidb-source-code-reading-3/)