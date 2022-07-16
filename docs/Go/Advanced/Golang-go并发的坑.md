```go
func main() {

    for i := 1; i <= 3; i++ {
    	go func(){
        	fmt.Println(i)
    	}(i)
    }
}
// 输出 3 1 2
```

原因，

runnext ：3

runq ：1 2

先执行 runnext，在执行 runq，所以结果是 3 1 2 

>   在 257 都是这个规律

因为本地 G队列 runq 容量只有 256，加上一个 runnext 里的，一共 257

超过 256 runq 里的 1 2 3 ... 就会被搬到 全局G队列了，然后在执行的时候并不是 runq 执行完采执行全局 G 队列里的，而是 runq 执行 61 个，就去执行几个全局 G 队列的。

[幼麟实验室-卷卷面试题](https://www.bilibili.com/video/BV19b4y1i74w?spm_id_from=333.999.0.0)

