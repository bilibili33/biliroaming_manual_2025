# 自建反代服务器  

https://github.com/ipcjs/bilibili-helper/pull/711#issue-542876230

其实我感觉看这个就够了

但是我想讲点阴招

因为首先反代必须要走443+ssl，但是如果你是要海外回国看番的话就会知道这个其实是个比较蛋疼的东西

看港澳台自建照着抄一下就行了，在对应地区买个vps装个nginx把配置抄上绑个域名套个证书就解决了，你甚至还能直接套cf，证书都不用管了

但是国内你要买vps搞443绑域名套证书的话要备案，烦上加烦了属于是

---

## 下面隆重介绍一下我的阴招（难绷），必备一条可以直接被访问的家宽，任一高位端口，套ddns，全球任一地区443可用vps一只

---

其实我感觉上面一句话就概括完了但还是详细讲一下吧

```
   ┌───────┐              _____                     |
   │───────│    443      |o____|    任一端口      /───\      
   │       │  ------>    |o____|   -------->    /      \    
   │       │             |o____|     无TLS     | O   |──|    
   └───────┘                                   |     |  |
     浏览器               服务器                   家宽

```
首先在家宽的机子上开一个nginx，配置如下

```nginx
server
{
    listen 51234;
    listen [::]:51234;  # 如果不需要ipv6监听的话这行可删

    client_max_body_size 128M;

    location /pgc/player/api/playurl { 
      proxy_pass https://api.bilibili.com;
    }

    location /pgc/player/web/playurl {
      proxy_pass https://api.bilibili.com;
    }
}
```

然后把端口放出去

之后在vps上新建一个站点，绑个域名，然后在默认给的配置上添加两个location（加在server块里）

```nginx
location /pgc/player/api/playurl {
  proxy_pass http://dir.114514486.xyz:50080/pgc/player/api/playurl;
  if ($http_x_from_biliroaming ~ "^$") {
    return 403;
  }
}
    
location /pgc/player/web/playurl {
  proxy_pass http://dir.114514486.xyz:50080/pgc/player/web/playurl;
}
```

其实如果你只是用web看的话上面两个代码块里面的location块中关于api的那个可以不写，那个是给客户端用的，网页只会去请求web的地址

这边如果你的域名是在cf，对应域名的ssl/tls是灵活的话，那直接把你帮好的域名填到代理服务器那边应该就可以用了

如果不是那就在vps内给域名申请个证书绑上去就行
