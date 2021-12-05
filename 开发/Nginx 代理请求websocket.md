在 spring boot 中提供一个websocket，前端通过websocket连接，但是经过前端特定前缀代理后。发现并不能访问到后台websocket。

### 假定

* http://127.0.0.1:8301/websocket 是可以访问到websocket的。
* 前端配置固定代理转发后台地址，比如： http://127.0.0.1/api 转发到 --> http://127.0.0.1:8301

　　如果这个时候前端需要连接到后台的websocket，需要使用

```
#常规地址转发到后台
location /api/ {
    proxy_pass   http://127.0.0.1:8301/;
	break;
}

#特殊websocket转发到后台
location /api/websocket {
    #关键是这里，需要重现url
    rewrite /api/websocket/(.*) /websocket/$1 break;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_pass http://127.0.0.1:8301;
}
```

> rewrite指令说明：rewrite regex replacement [flag];
>
