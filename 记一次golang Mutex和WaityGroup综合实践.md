#### 记一次golang Mutex和WaityGroup综合实践
* 场景：看到锁的用法时，想到会有多个协程同时对一个共享变量进行操作的情景，于是就想用一个例子来实现这个场景
* 问题1：如何保证多个协程都能够完全执行完？
* 解决：
	1. 在主协程中加上time.Sleep()，使主协程睡眠，但不可估计程序执行的时间，睡眠时间不能精确
	2. 使用通道，在每个协程中向通道发送数据，在主协程中对每次的发送进行接收，但这样反复申请通道会造成很大的内存开销
	3. 使用waitGroup，在主协程的一开始就将wg.add()设置为协程的数量，并使用wg.wait进行等待，主协程开始阻塞，然后开启各个协程，在每个协程结束后进行wg.done，即可保证每个协程都执行完成
* 问题2：在程序中，多个方法对同一个变量num进行赋值操作，并在赋值后进行打印，如图，如果在赋值后，将程序睡眠一段时间，此时如果还有其他的协程未执行，便会在睡眠时间修改了num的值，当睡眠完毕后打印的就不是这个协程中修改的值，会影响到之后的动作
```
func addNum1()  {
	num = 1
	time.Sleep(time.Second*5)
	fmt.Println(num)
	wg.Done()
}
```
* 解决：此时为了保护变量访问原子性，要使用锁sync.Mutex
	* 问题：在给num赋值1的协程中加锁，预想情况是不会再出现1被其他协程修改的情况，但问题仍然未解决，debug中发现锁确实生效了，但其他协程并没有等待协程1做完所有的动作，此时便有了一个问题，锁并没有锁住整个协程的动作，而是相对其他人获得了锁，其他人若不想获得锁，便没有竞争关系，也无法约束，当每个人都想获得锁时，别人才会去等待你把锁使用完。
	* 解决：在每个协程中都进行锁动作，这样每个协程只要抢到锁，其他都会等待他执行完毕，如果只给某一个协程上锁，其他协程对锁不关心，锁便达不到保护效果
	

> 思考： 由上总结，锁的使用方法是，让想参与变量修改的协程都上锁，谁抢到锁，谁就可以进行修改并不受他人影响，锁并不保护某个协程享有修改变量的权力，权力是公平的，锁只保证在协程获得权力后不被其他协程影响，换言之，锁不是为保护协程的动作而生的，而是保护变量的动作而生的，这其中是不是也有面向对象的思想呢？不能因为想保护某个协程的修改动作而去试用锁，若从变量角度出发，变量不能被随意修改，应加锁约束所有修改人的行为，而不是给某个动作加锁，约束其他人的行为。