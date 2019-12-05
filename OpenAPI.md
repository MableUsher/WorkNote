#### OpenAPI
1. 类型 三种
    1. 数据型
    2. 应用型
    3. 资源型
2. 基于Http协议开发，请求为标准的Http请求，返回使用Xml，Json
3. url
    1. schemes：协议名
    2. host：主机名
    3. basepath：根路径
4. api操作
    1. path：路径，访问一个资源
    2. get：http方法名，用来操作动词
        1. summary：描述信息
        2. description：说明该方法
        3. responses：定义响应类型
            * 200 http状态码
                1. description：描述信息
                2. schme：定义响应内容
5. 添加分页参数（query paramater）
```
parameters:
    - name: pageSize
    in: query
    description: Number of persons returned
    type: integer
    - name: pageNumber
    in: query
    description: Page number
    type: integer
```

6. 定义路径参数（path paramater ）
```
/persons/{username}
parameters:
    - name: username
    in: path
    required: true
    description: The person's username
    type：string
```

7. post方法
    1. 定义消息体参数（body paramater）
        in：body
    2. 定义状态
        * 204：成功创建
        * 400：创建失败

8. definition
    * 一处定义多处调用
    * 一般用来定义cheme中可重用的属性
    * 定义过程一般放在文档末尾
    
```
 $ref: "#/definitions/Person"
```

9. 路径级别参数的定义则可以直接放在方法前统一定义，如：
```
/persons/{username}:
    parameters:
        - name: username
        in: path
        required: true
        description: The person's username
        type: string
```

10. 定义字符串
11. 日期和时间
12. 数字类型和范围
13. 枚举类型
14. 数值的大小和唯一性
15. 读写操作同一定义的数据（只读属性）
16. 参数定义
    1. 必带参数及属性：required
    2. 默认参数：default
    3. 空值：allowEmptyValue
17. 响应消息头的更改是否支持definition
18. 分类标签（Tags）




