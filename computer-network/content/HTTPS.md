# HTTP与HTTPS的区别
- HTTP是超文本传输协议，信息是明文进行传输，存在安全风险的问题，HTTPS在TCP和HTTP之间加了SSL/TLS安全协议，使得明文能够加密传输
- HTTP连接建立相对简单，TCP三次握手后便可进行HTTP的报文传输。而HTTPS在TCP三次握手后，还需要进行SSL/TLS的握手过程，才可以进入加密报文传输
- HTTP的端口是80，HTTPS的端口是443
- HTTPS协议需要向CA(证书权威机构)申请数字证书，来保证服务器的身份是可信的

