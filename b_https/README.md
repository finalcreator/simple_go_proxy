# A. simple
1. openssl genrsa -out server.key 2048
2. openssl req -new -x509 -key server.key -out server.crt -days 365




# B. self-signed(自签发)
(1)openssl genrsa -out rootCA.key 2048
(2)openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=*.tunnel.tonybai.com" -days 5000 -out rootCA.pem

(3)openssl genrsa -out device.key 2048
(4)openssl req -new -key device.key -subj "/CN=*.tunnel.tonybai.com" -out device.csr
(5)openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000

(6)cp rootCA.pem assets/client/tls/ngrokroot.crt
(7)cp device.crt assets/server/tls/snakeoil.crt
(8)cp device.key assets/server/tls/snakeoil.key

自己搭建ngrok服务，客户端要验证服务端证书，我们需要自己做CA，因此步骤(1)和步骤(2)就是生成CA自己的相关信息。
步骤(1) ，生成CA自己的私钥 rootCA.key
步骤(2)，根据CA自己的私钥生成自签发的数字证书，该证书里包含CA自己的公钥。

步骤(3)~(5)是用来生成ngrok服务端的私钥和数字证书（由自CA签发）。
步骤(3)，生成ngrok服务端私钥。
步骤(4)，生成Certificate Sign Request，CSR，证书签名请求。
步骤(5)，自CA用自己的CA私钥对服务端提交的csr进行签名处理，得到服务端的数字证书device.crt。

步骤(6)，将自CA的数字证书同客户端一并发布，用于客户端对服务端的数字证书进行校验。
步骤(7)和步骤(8)，将服务端的数字证书和私钥同服务端一并发布。



# C. 客户端对服务端数字证书进行验证（gohttps/5-verify-server-cert）

> 首先我们来建立我们自己的CA，需要生成一个CA私钥和一个CA的数字证书:
1. openssl genrsa -out ca.key 2048
2. openssl req -x509 -new -nodes -key ca.key -subj "/CN=tonybai.com" -days 5000 -out ca.crt

> 接下来，生成server端的私钥，生成数字证书请求，并用我们的ca私钥签发server的数字证书：
3. openssl genrsa -out server.key 2048
4. openssl req -new -key server.key -subj "/CN=localhost" -out server.csr
5. openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 5000

> 现在我们的工作目录下有如下一些私钥和证书文件: 
* CA:
    * 私钥文件 ca.key
    * 数字证书 ca.crt

* Server:
    * 私钥文件 server.key
    * 数字证书 server.crt