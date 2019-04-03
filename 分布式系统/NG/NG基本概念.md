### NG的作用
    正向代理
    反向代理
    负载均衡

#### 正向代理
    https://www.cnblogs.com/Anker/p/6056540.html
#### 反向代理
    https://www.cnblogs.com/Anker/p/6056540.html

#### 负载均衡

### 安装NG
    下载，运行nginx.exe
    修改nginx.conf
### 修改NG配置
```
    main # 全局配置
    events { # nginx工作模式配置
    }
    http { # http设置
        server { # 服务器主机配置
            location { # 路由配置
            }
            location path {
            }
            location otherpath {
            }
        }
        server {
            location {
            }
        }
    upstream name { # 负载均衡配置
        }
    }
```
    1.main：用于进行nginx全局信息的配置
    2.events：用于nginx工作模式的配置
    3.http：用于进行http协议信息的一些配置
    4.server：用于进行服务器访问信息的配置
    5.location：用于进行访问路由的配置
    6.upstream：用于进行负载均衡的配置

### NG负载均衡
    根据请求的url的productNo进行hash，把不同的productNo的请求分发到不同的服务器上