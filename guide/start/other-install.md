# 其他服务部署

## 插件编译

如果要使用插件，需要提前将插件进行编译。直接使用plugins下面的编译脚本直接执行就可以。

## 服务端编译部署

一、流媒体部署

1. 进入项目根目录下，使用`./build.sh linux|windows|mac`进行程序编译
2. 进入到服务器`/opt/sagoo/iot-server`下
3. 创建server文件夹及SagooMedia子文件夹，并将第一步编译后的所有文件放到SagooMedia子文件夹下
4. 进入到`server/SagooMedia`目录下修改config.toml配置，可参考以下配置内容

      ```toml
         [Engine]
        EnableAudio = true
        EnableVideo = true
        # 发布流默认过期时间单位秒
        PublishTimeout = 60
        # 自动关闭触发后延迟的秒数(期间内如果有新的订阅则取消触发关闭)
        AutoCloseAfter = 10
        # RTP包乱序重排
        RTPReorder = false
        [Summary]
        # 1秒中采样一次
        SampleRate = 1
        [RTMP]
        ListenAddr = ":1935"
        [GateWay]
        ListenAddr = ":8166"
        #ListenAddrTLS = ":8082"
        #CertFile = "server.pem"
        #KeyFile = "server.key"
        StaticPath = "./public"
        [Jessica]
        #ListenAddr = ":8081"
        #ListenAddrTLS = ":8083"
        #CertFile = "xxx.cert"
        #KeyFile = "xxx.key"
        [LogRotate]
        # 日志存储目录相对或绝对
        Path = "logs"
        # 日志是否按大小分割，0表示不按大小分割，非零代表按该大小字节进行分割
        Size = 0
        Days = 1
        # 按照go layout格式化，默认按照小时
        Formatter = "2006-01-02T15"
        # [FFMPEG]
        [HLS]
        # 是否开启写磁盘，开启后侦测到发布流就会开始写TS文件
        EnableWrite = false
        # 是否打开内存模式，在内存中保留TS数据，方便直接读取
        EnableMemory = false
        # 分片大小 单位秒
        Fragment = 10
        # 窗口数里，代表一个m3u8文件里面有几个ts
        Window = 2
        # ts文件存放目录，m3u8会存放在上一级
        Path = "resource"
        [HDL]
        #ListenAddr = ":2020"
        #ListenAddrTLS = ":2021"
        #CertFile = "xxx.cert"
        #KeyFile = "xxx.key"
        #Reconnect = true
        [HDL.AutoPullList]
        # "live/hdl" = "http://flv.bdplay.nodemedia.cn/live/bbb.flv"
        [TS]
        # ts存放目录
        Path  = "resource"
        [Record]
        Path = "resource"
        # 自动录制功能
        AutoRecord  = false
        [RTSP]
        # 端口接收推流
        ListenAddr = ":554"
        Reconnect = true
        #启动后自动拉流，可以配置多个
        [RTSP.AutoPullList]
        # "live/rtc" = "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4"
        # "live/rtsp" = "rtsp://admin:123456@42.193.7.166:9018/video1"
        [WebRTC]
        # 端口范围不配置的话是自动分配
        # PortMin = 30000
        # PortMax = 40000
        # 公网访问必须配置PublicIP，否则无法建立连接
        # PublicIP = ["192.168.1.120"]
        # WebRTC 推流时控制GOP大小，单位毫秒
        # PLI = 2000
        [GB28181]
        Serial = "34020000002000000001"
        Realm = "3402000000"
        Expires = 3600
        # 媒体端口
        # MediaPort = 58200
        # 开启TCP拉流，默认关闭
        # TCP = true
        # TCP端口数量，超过一个的话将会每个设备轮流使用，从MediaPort往下递增
        # TCPMediaPortNum = 1
        ListenAddr = "192.168.1.120:5060"
        # 自动停止发布，当订阅者数量将为0时，延迟N秒自动断开,-1代表不断开
        AutoCloseAfter = -1
        # 自动拉流，如果开启，则拿到设备注册信息后，就从设备拉流
        AutoInvite = true
      ```

      其中服务端口号需根据实际情况进行修改
      **如需开启SSL，则需将一下可按照一下步骤进行开启**
        1.  注释`ListenAddr = ":8166"`配置
        2. 放开以下配置（证书及秘钥需根据实际情况修改路径）

      ```toml
        #ListenAddrTLS = ":8166"
        #CertFile = "server.pem"
        #KeyFile = "server.key"
      ```

5. 使用`./curl.sh start`进行启动

二、rule-engine(规则引擎)部署

**服务器(cent os)安装nodejs 最新版**

1. 查看是否为最新版本

```shell
  yum list nodejs
``` 

2. 如果不是，通过下面代码增加源信息

```shell
  curl --silent --location https://rpm.nodesource.com/setup_18.x | sudo bash
```

3. 安装nodejs

```shell
    yum install nodejs
```

**如果安装包过慢，可以先设置淘宝源  

```shell
  npm config set registry https://registry.npm.taobao.org/
```

4. 全局安装 pm2 

```shell
  sudo npm i pm2 -g
 ```

5. 将【rule-engine】项目拷贝到服务器上

6. 在【rule-engine】目录下安装包依赖

   ```shell
    npm install
    ```

7. 启动项目 

```shell
    pm2 start packages/node_modules/node-red/red.js --name rule-engine:2881
```

之后可以通过 `pm2 show rule-engine:2881` 查看项目运行情况

8. 按照【rule-engine】项目下 `nginx/node-red.conf` 文件的配置增加一下nginx配置，来保证规则引擎和iot的同源

9. 如果要修改favicon.ico, 就将`packages/node_modules/@node-red/editor-client/public/favicon.ico`这个文件进行替换即可。

## 前端编译部署

一、大屏项目部署

1. 进入项目根目录下使用`npm run build`命令进行源码编译

2. 进入`/opt/sagoo/iot-ui`目录下创建plugin文件夹
3. 将编译之后生成的screen全部文件上传到服务器`/opt/sagoo/iot-ui/plugin`目录下

二、组态项目部署

1. 进入项目根目录下使用`npm run build`命令进行源码编译

2. 进入`/opt/sagoo/iot-ui`目录下创建plugin文件夹
3. 将编译之后生成的topo全部文件上传到服务器`/opt/sagoo/iot-ui/plugin`目录下

## nginx配置

1. 修改nginx配置，可参考以下配置内容

   ```nginx
      server {
        listen       80;
        server_name  localhost;
        
        #ssl_certificate /data/nginx/conf/vhost/cert/localhost/fullchain.pem;     
        #ssl_certificate_key /data/nginx/conf/vhost/cert/localhost/privkey.pem;

        #前端路由
        location / {
            root /opt/sagoo/iot-ui;
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
      

        #sagoo-iot路由
        location /base-api/ {
                  proxy_set_header Connection upgrade; 
                  proxy_set_header Upgrade $http_upgrade;
                  proxy_http_version 1.1; 
                  chunked_transfer_encoding off; 
                  proxy_pass                 http://127.0.0.1:8200/;
                  proxy_redirect             off;
                  proxy_set_header           Host             $host;
                  proxy_set_header           X-Real-IP        $remote_addr;
                  proxy_set_header           X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
      
        #接口文档路由
        location /base-api/swagger/api.json {
            proxy_pass                 http://127.0.0.1:8200/base-api/swagger/api.json;
        }

        #规则引擎路由
        location /rule-engine  {
                  proxy_pass http://127.0.0.1:2881/rule-engine;
                  autoindex on;
                  autoindex_exact_size on;
                  autoindex_localtime on;
        }
        
        location /rule-engine/comms {
                  proxy_pass http://127.0.0.1:2881;
                  proxy_read_timeout 300s;
                  proxy_set_header Host $host;
                  proxy_set_header X-Real-IP $remote_addr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                  proxy_http_version 1.1;
                  proxy_set_header Upgrade $http_upgrade;             
                  proxy_set_header Connection "upgrade";
        }

        #流媒体服务路由
        location /media/ {
              proxy_http_version 1.1;
              proxy_set_header Upgrade $http_upgrade;
              proxy_set_header Connection upgrade;
              proxy_pass    http://127.0.0.1:8166/;
              proxy_redirect             off;
              proxy_set_header           Host             $host;
              proxy_set_header           X-Real-IP        $remote_addr;
              location ~* \.(jpg|jpeg|png|ico|css|js)$ 
              {
                    rewrite ^/media/(.*)$ /$1 break;
                    root /opt/sagoo/iot-server/server/SagooMedia/public;
                }

            }
       }
   ```

2.  进入到`/usr/local/nginx/sbin`目录下，使用`./nginx -s reload`重载nginx配置文件


部署完可以通过`http://localhost`进行访问, 超级管理员账号: `admin` 密码: `admin123456`

### 其它问题

一、**在macOS下遇到 Warning :`IOMasterPort`：**

```shell
warning: ‘IOMasterPort‘ is deprecated: first deprecated in macOS 12.0 [-Wdeprecated-declarations]
```

**原因**

依赖包跟MacOS的版本有兼容问题。

解决方案
切换CGO编译方式

```shell
go env -w CGO_ENABLED="0"
```

二、**访问过程中，发现视频监控下的菜单打不开，可参考以下步骤进行排查：**

1. 进入`/opt/sagoo/iot-server/bin/server/SagooMedia/var`目录下，使用`tail -f`查看服务是否启动成功
2. 使用超级管理员账户进行登录，打开系统配置-菜单管理，找到物联管理-视频监控查看对应的菜单链接地址是否为当前域名和端口号，更改完需退出重新登陆方可生效。
3. 查看nginx配置，检查流媒体路由是否配置正确(ip、端口号)

三、**访问过程中，发现规则引擎-规则编排的菜单里面的规则编辑打不开或者点开跳转的是登录页面，可参考以下步骤进行排查：**

1. 检查规则引擎是否启动成功，开放端口是否跟nginx配置转发路由端口一致
2. 查看`/opt/sagoo/iot-server/bin/sagoo-admin`是否以8200端口进行启动