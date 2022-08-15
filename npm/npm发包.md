发包需要准备的环境

+ node版本（npm版本）不能过低
+ npm源必须指向 npm官方镜像 
+ 2021年10月4日以后，npm 网站和 npm 注册表你必须使用 TLS 安全套接字层1.2版，所以你需要升级相应的版本

解决办法

```
npm install -g https://tls-test.npmjs.com/tls-test-1.0.0.tgz
npm config set registry https://registry.npmjs.org
```

