# laravel-socket.io

### 1.先查看自己的環境，本環境使用了 windows & docker-desktop & laradock    
如果是vagrant或是其他linux環境下架設    
參考 => https://learnku.com/laravel/t/13101/using-laravel-echo-server-to-build-real-time-applications  

### 2.laradock 架設本地環境   
參考 => https://github.com/wanderingRock/WindowsLaradock

### Step.1 
### 安裝 Redis
--- 
Redis 的要調整設定黨   
1.修改 .env 環境變數 
確認REDIS_PORT 
```
REDIS_PORT=6379  //看情況自行設定
```

2.確認路徑 laradock\redis 下的 Dockerfile 檔案與設定的Port一致
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

3.修改路徑 laradock\redis 下的 redis.conf 檔案   
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

參考 => https://hoohoo.top/blog/laradock-redis-production-environment-configuration/

### Step.2  安裝 laravel-echo-server
1.下安裝指令      
當然有相關設定可以至   
路徑 laradock\laravel-echo-server 下的 laravel-echo-server.json 修改設定值 (這裡目前我沒特別設定)
```
docker-compose up -d laravel-echo-server
```
確認是否安裝可以看 docker ui 介面

### Step.3  進入環境並且確認相關套件

1進入docker容器 名為 workspace 容
```
docker exec -it laradock_workspace_1 bash  
winpty docker exec -it laradock_workspace_1 bash
```

2.確認是否有composer & npm & node
```
composer -v
npm -v
node -v
```
可先將上述幾項都更新至最新    
我這邊只作 npm update 因為其他已為最新   
```
npm update
```
### Step.4  建立專案
1.建立專案 
```
composer create-project --prefer-dist laravel/laravel laravel
cd laravel
```

2.安裝Predis
```
composer require predis/predis
```

3.安裝node_modules
```
npm install
```

4.於/config/app.php 備註 App\Providers\BroadcastServiceProvider::class 這行
```
//App\Providers\BroadcastServiceProvider::class,
```

5.於/config/app.php 取消備註 // 'Redis' => Illuminate\Support\Facades\Redis::class,
```
'Redis' => Illuminate\Support\Facades\Redis::class,
```

6.修改 .env 的 關於redis的配置
```
BROADCAST_DRIVER=redis
QUEUE_CONNECTION=redis
REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379
REDIS_CLIENT=predis //新增
```
7.對於專案安裝 socket.io-client & laravel-echo
```
npm install --save socket.io-client@2.4.0 //最新版本有相關錯誤
npm install --save laravel-echo
```
### Step.5  開始撰寫

1.於 webpack.mix.js 新增以下程式碼
```
mix.options({
    legacyNodePolyfills: true
});
```
原因請參考 => https://laravel-mix.com/docs/6.0/legacy-node-polyfills
