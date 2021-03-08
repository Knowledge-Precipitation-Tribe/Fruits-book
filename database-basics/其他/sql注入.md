# 什么是sql注入
sql注入攻击是通过将恶意的sql查询或添加语句插入到应用的输入参数中，在后台服务器上解析执行进行的攻击，它目前黑客对数据库进行攻击的最常用手段之一。

# 为什么会产生sql注入
- WEB开发人员无法保证所有的输入都已经过滤
- 攻击者利用发送给SQL服务器的输入参数构造可执行的SQL代码（可加入到get请求、post请求、http头信息、cookie中）
- 数据库未做相应的安全配置

# 常见的sql注入方式
## 数字注入
在浏览器地址栏输入：learn.me/sql/article.php?id=1，这是一个get型接口，发送这个请求相当于调用一个查询语句：
```
$ sql = "SELECT * FROM article WHERE id =",$id
```
正常情况下，应该返回一个id=1的文章信息。那么，如果在浏览器地址栏输入：learn.me/sql/article.php?id=-1OR1=1，这就是一个SQL注入攻击了，可能会返回所有文章的相关信息。为什么会这样呢？

这是因为，id = -1永远是false，1=1永远是true，所有整个where语句永远是ture，所以where条件相当于没有加where条件，那么查询的结果相当于整张表的内容

## 字符串注入
有这样一个用户登录场景：登录界面包括用户名和密码输入框，以及提交按钮。输入用户名和密码，提交。

这是一个post请求，登录时调用接口learn.me/sql/login.html，首先连接数据库，然后后台对post请求参数中携带的用户名、密码进行参数校验，即sql的查询过程。假设正确的用户名和密码为user和pwd123，输入正确的用户名和密码、提交，相当于调用了以下的SQL语句：
```
SELECT * FROM user WHERE username = 'user' ADN password = 'pwd123'
```
由于用户名和密码都是字符串，SQL注入方法即把参数携带的数据变成mysql中注释的字符串。mysql中有2种注释的方法：

**1）'#'**
'#'后所有的字符串都会被当成注释来处理

用户名输入：user'#（单引号闭合user左边的单引号），密码随意输入，如：111，然后点击提交按钮。等价于SQL语句：
```
SELECT * FROM user WHERE username = 'user'#'ADN password = '111'
```
'#'后面都被注释掉了，相当于：
```
SELECT * FROM user WHERE username = 'user' 
```

**2）'-- ' （--后面有个空格）**
'-- '后面的字符串都会被当成注释来处理

用户名输入：user'-- （注意--后面有个空格，单引号闭合user左边的单引号），密码随意输入，如：111，然后点击提交按钮。等价于SQL语句：
```
SELECT * FROM user WHERE username = 'user'-- 'AND password = '111'
```
'-- '后面都被注释掉了，相当于：
```
SELECT * FROM user WHERE username = 'user'
```
因此，以上两种情况可能输入一个错误的密码或者不输入密码就可登录用户名为'user'的账号，这是十分危险的事情。

# 如何防止sql注入
- 页面静态化
- 使用参数化的查询
- 严格检查输入变量的类型和格式
- 过滤和转义特殊字符