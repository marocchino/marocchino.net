---
title: BitWarden(rs)를 라즈베리파이에 설치하기
date: 2021-04-17 12:25:40
tags:
---


# 과정

## subdomain ipv6으로 설정

젤 오래걸리니 젤 먼저하는 걸 추천 v4는 A고 v6는 AAAA인것만 기억하면 어려울것 없다.
주소는 ifconfig로 확인가능. (아마 없을테니까 apt로 깔자)

## Docker

```sh
sudo apt install docker docker-compose
sudo adduser bitwarden
sudo groupadd docker
sudo usermod -aG docker bitwarden
```

## Let's Encrypt

난 여기서 파일 권한 문제로 삽질 많이했는데 그냥 bitwarden계정으로 도커를
실행하면 알아서 퍼미션 잡힐테니 그냥 이렇게 하면 된다.

```sh
sudo su - bitwarden
docker run -it --rm --name certbot \
  -v "/etc/letsencrypt:/etc/letsencrypt" \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  certbot/certbot:arm64v8-latest certonly \
  --standalone -d $YOUR_DOMAIN_HERE -m $YOUR_EMAIL_HERE --agree-tos
```

## 서버 띄우기

`start.sh`파일을 만들어두자.
로그볼일이 있을까 싶어서 일부러 `--rm`옵션은 주지 않았다.
ROCKET_ADDRESS는 디폴트가 0.0.0.0이라 그대로두면 IPv4에서 밖에 인식 못하니 주의

<!-- markdownlint-disable MD013 -->
```sh
#!/bin/bash
yes | docker container prune # prune이 싫으면 docker rm bitwarden 해도 된다
docker run -d --name bitwarden \
  -e ROCKET_TLS="{certs=\"/letsencrypt/live/$YOUR_DOMAIN_HERE/fullchain.pem\",key=\"/letsencrypt/live/$YOUR_DOMAIN_HERE/privkey.pem\"}" \
  -e ROCKET_ADDRESS='::' \
  -e ADMIN_TOKEN="$ADMON_TOKEN" \
  --hostname your.servername.here \
  -v /etc/letsencrypt/:/letsencrypt/ \
  -v /home/bitwarden/bwdata/:/data/ \
  -p 443:80 \
  vaultwarden/server:latest
```
<!-- markdownlint-enable MD013 -->

## 인증서 자동 갱신

`renew.sh`파일도 만들자.

```sh
#!/bin/bash
yes | docker container prune
docker run -it --name certbot \
  -v "/etc/letsencrypt:/etc/letsencrypt" \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  certbot/certbot:arm64v8-latest renew
```

매월 15일에 갱신하고 내친김에 리붓했을때 서버 자동으로 켜게 해두자.

```sh
crontab -u bitwarden -e
00 04 15 * * /home/bitwarden/renew.sh
@reboot /home/bitwarden/start.sh
```

## IPv6 IP 고정

라즈비안에서는 그런 일없었는데 리붓할때 마다 아이피가 바뀐다.

Network -> Wired -> Config -> IPv6가서 다음 과 같이 설정한다.
v6 IP는 대충 `2409:10:12:34:56:78:90:12`라고 가정해보자.

Method: Manual
Address: 2409:10:12:34:56:78:90:12
Prefix: 2409:10:12:34::/64
Gateway: 2409:10:12:34::1

리붓하면 아이피가 고정된다.

## Todo

갱신이 7월 16일이니 그때 갱신 됐는지 확인하면 될듯.

## 이력

- 2021.05.22 IP 고정항목 추가, bitwardenrs -> vaultwarden, ROCKET_ADDRESS 추가
