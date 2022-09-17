**<span style="font-size: 35px;">🍮 JavaWeb</span>**

---

### 下面有关forward和redirect的描述，正确的是？

![forward和redirect的描述](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-27/ca5cc336-df08-4644-b22f-6a0d560f63c7.png)
**request的forward和response的redirect**

1. redirect地址栏变化，forward发生在服务器端内部从而导致浏览器不知道响应资源来自哪里
2. redirect可以重定向到同一个站点上的其他应用程序中的资源，forward 只能将请求 转发给同一个WEB应用中的组件
3. redirect默认是302码，包含两次请求和两次响应
4. redirect效率较低