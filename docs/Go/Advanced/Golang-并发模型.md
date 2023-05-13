# 并发模型

## MPSC (Multiple Producers and Single Customer) 多生产单消费模型

### 共用一个 channel
统一在 main 控制，先把 consumer 跑起来，这时候它会一直在那等待。
然后开启 producer，让 producer 完成任务之后调用 wg.Done().
开启完所有 producer 之后就 Wait 等待所有生产者完成后 close 掉 channel。
这样 consumer 消费完了就退出了。
```go
func producer(wg *sync.WaitGroup, sendChan chan string) {
    for i := 0; i < 10; i++ {
        sendChan <- "prod"
    }
    wg.Done()
}

func consumer(sendChan chan string) {
    for v := range sendChan {
        fmt.Println(v)
    }
}

func main() {]
    ch := make(chan string)

    wg := sync.WaitGroup{}
    wg.Add(2)
    go producer(wg, sendChan)
    go producer(wg, sendChan)
    
    go func() {
        wg.Wait()
        close(sendChan)
    }()

    consumer(sendChan)
}
```

### 每个生产者一个 channel
```go
func producer(in []int) chan Msg {
    ch := make(chan Msg)
    go func() {
        for _, v := range in {
            ch <- Msg{in: v}
        }
        close(ch)
    }()
    return ch
}

func consumer(ch1, ch2 chan Msg) {
    var v1 Msg
    var v2 Msg
    ok1 := true
    ok2 := true

    for ok1 || ok2 {
        select {
        case v1, ok1 = <-ch1:
            fmt.Println(v1)
            if !ok1 { //通道关闭了
                ch1 = nil
            }
        case v2, ok2 = <-ch2:
            fmt.Println(v2)
            if !ok2 { //通道关闭了
                ch2 = nil
            }
        }
    }
}

func main() {
    ch1 := producer([]int{1, 2, 3})
    ch2 := producer([]int{4, 5, 6})

    consumer(ch1, ch2)
}
```

## MCSP (Multiple Customers and Single Producer) 多消费单生产模型
只有一个生产者，那么使用一个 channel 就够了。所以由生产者关闭 channel 即可
```go
func producer(sendChan chan int) {
    for _, v := range []int{1, 2, 3, 4, 5, 6, 7, 8, 9} {
        sendChan <- v
    }
    close(sendChan)
}

func consumer(wg *sync.WaitGroup, sendChan chan int) {
    for v := range sendChan {
        fmt.Println(v)
    }
    wg.Done()
}

func main() {
    ch := make(chan int)
    go producer(ch)
    wg := sync.WaitGroup{}
    wg.Add(2)
    go consumer(wg, ch)
    go consumer(wg, ch)
    wg.Wait()
}
```

## MPMC (Multiple Producers and Multiple Customer) 多生产多消费模型
