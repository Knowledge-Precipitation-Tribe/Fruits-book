session

服务端保持状态和数据的方案

## [[cookie]]与session的关系
### 区别
1. cookie数据保存在用户浏览器上，session数据保存在服务器上
2. session保存在服务器上，当访问量较大时会比较占用资源
3. 单个cookie保存的数据不能超过4k，session没有限制

### 联系


## 分布式session的实现原理