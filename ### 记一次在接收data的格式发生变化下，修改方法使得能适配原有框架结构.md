### 记一次在接收data的格式发生变化下，修改方法使得能适配原有框架结构

![1568624399892](C:\Users\pzy\AppData\Roaming\Typora\typora-user-images\1568624399892.png)

* 如图所示，方法中接受的是string类型的数据，现有方法为

  ![1568624494897](C:\Users\pzy\AppData\Roaming\Typora\typora-user-images\1568624494897.png)

* 问题1：如果在handle中直接调用，则由于数据类型不匹配报错，
* 解决：方法中接收的是string数组，现在就要做一个转化，把json格式的数组转化成string数组，就要引入json包中的Unmarshal方法
```
func Unmarshal(data []byte, v interface{}) error
```
* 先要将string数据转化为字节数组，然后将想要转化成的容器形态传入，便可以自动填充，所以此时需要制造一个字符串数组容器
```
type IpsecSaNames struct {
    SaNames []string `json:"saNames"`
}
```
* 主要的阻塞就在于这个结构体，开始在看到接口类型时，不知道应该怎样定义，这个接受体还必须要传递指针才可以进行正常填充，所以应该像下面的代码这样
```
ipsecSaNames := &IpsecSaNames{}
err := json.Unmarshal([]byte(data),ipsecSaNames)
if err != nil {
	return "",err
}
```
* 先定义一个结构体指针，再将其传入，再把转化出的结构体类型转化为json返回给上层，即可与之前handle方法完美适配