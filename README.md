# laravel-socket.io

### 1.先查看自己的環境，本環境是使用laradock架設    
如果是vagrant或是其他linux環境下架設    
參考 => https://learnku.com/laravel/t/13101/using-laravel-echo-server-to-build-real-time-applications  

### 2.laradock 架設本地環境   
參考 => https://github.com/wanderingRock/WindowsLaradock

### Step.1 
### 安裝 Redis & laravel-echo-server
--- 
Redis 的要調整設定黨   
1.修改 .env 環境變數 
確認REDIS_PORT 
```
REDIS_PORT=6379  //看情況自行設定
```

2.確認路徑laradock/redis 的 Dockerfile 檔案與設定的Port一致
```
FROM redis:latest

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"

## For security settings uncomment, make the dir, copy conf, and also start with the conf, to use it
RUN mkdir -p /usr/local/etc/redis
COPY redis.conf /usr/local/etc/redis/redis.conf

VOLUME /data

EXPOSE 6379

CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
#CMD ["redis-server"]
```

3.修改路徑為 laradock/redis 的 redis.conf 檔案   
-註解 bind 127.0.0.1
```
# bind 127.0.0.1
```
-將 protected-mode 改為 no
```
protected-mode no
```
-設置連線密碼，解除 requirepass 註解，設定密碼 (此步驟沒有想設定密碼可不設置)
```
# quirepass 你的密碼 //要設定請把#拿掉
```
4.重置 Redis 環境    
停止且重新安裝 redis   
```
docker-compose stop redis
docker-compose build --no-cache redis
```
5.重新啟動
```
docker-compose up -d redis
```

參考 => ttps://hoohoo.top/blog/laradock-redis-production-environment-configuration/
