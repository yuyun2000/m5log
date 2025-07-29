



{
  "registry-mirrors": [
    "https://docker.xuanyuan.me",
    "https://docker.1panel.live"
  ]
}

或访问
https://jiasu.20010327.xyz/


前缀：
hub.kesry.eu.org/

m.daocloud.io/docker.io



sudo docker run -d \
  --name frps \
  --restart=always \
  -v ~/frp/frps.ini:/frp/frps.ini \
  -p 7000:7000 \
  -p 7500:7500 \
  -p 8080:8080 \
  hub.kesry.eu.org/fatedier/frps \
  -c /frp/frps.ini