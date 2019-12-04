### 记一次运用对wg的理解，修改论坛上对《Go语言圣经》9.1转账案例练习的解答
* 背景：在学习对共享变量的操作时，看到圣经上有一个转账案例的练习，在阅读之后发现了一些问题，于是进行了修改
```
// Package bank provides a concurrency-safe bank with one account.
package bank

var deposits = make(chan int) // send amount to deposit
var balances = make(chan int) // receive balance

func Deposit(amount int) { deposits <- amount }
func Balance() int       { return <-balances }

func teller() {
    var balance int // balance is confined to teller goroutine
    for {
        select {
        case amount := <-deposits:
            balance += amount
        case balances <- balance:
        }
    }
}

func init() {
    go teller() // start the monitor goroutine
}
```
* 场景：代码中定义了“转账”Deposi()t和“账户余额”Balance()两个函数，在初始版本中，没有用monitor协程来限制具体的转账行为，导致在同时对余额进行操作时，A操作的转账发生在B操作的转账中间，在读取B的金额和写入B的金额的间隙发生了A的转账，则系统不能够正确反馈给A正确的余额，会导致A的钱丢失。<br/> 现在通过通道将转账这个行为限制在一个协程中，无论有多少次并发转账，单次的转账行为是原子性的，这就是利用了通道的阻塞机制
* 练习要求：给程序添加一个Withdraw(amount int)取款函数。其返回结果应该要表明事务是成功了还是因为没有足够资金失败了。这条消息会被发送给monitor的goroutine，且消息需要包含取款的额度和一个新的channel，这个新channel会被monitor goroutine来把boolean结果发回给Withdraw。
* 原练习：
```
//取款用函数
func Withdraw(amount int)bool{
    Deposit(-amount)
    if Balance() < 0 {
        Deposit(amount)
        return false // insufficient funds
    }
    return true
}
//测试
    bank.Init()

    var wg sync.WaitGroup
    wg.Add(1)
    // Alice:
    go func() {
        defer wg.Done()
        bank.Deposit(200)                // A1
    }()
    wg.Add(1)
    // Bob:
    go func(){
        defer wg.Done()
        bank.Deposit(100)
    }()

    wg.Add(1)
    go func(){
        defer wg.Done()
        res := bank.Withdraw(200)
        if res{
            fmt.Println("取款成功")
        }else {
            fmt.Println("取款失败")
        }
    }()
    wg.Wait()

    res := bank.Balance()
    fmt.Printf("剩余:%d" , res)
}
```
* 分析：对于withdraw函数的解答没有大致的问题，函数满足了进行取款的要求并返回了取款结果成功与否，还考虑到了取款失败恢复账户余额，但其中的细节还是存在许多问题
* 问题1：取款逻辑为先进行取款操作，根据余额的正负情况判断取款是否成功，这里有点不符逻辑，在一件事情尚未发生的的时候应该先进行评估，应该先把存款数额与取款数额相比对，如果存款小于取款金额，直接返回取款失败，这样一是防止余额出现为负这种情况，二是避免了反复对账户进行操作，增加不必要的风险
* 问题2：在测试时，将两个存款行为和取款行为同时开启，存在的问题是可能取款先竞争到了执行权，那么结果不能保证一致性，也不易设定取款金额来查看结果
* 解决：应保证所有存款