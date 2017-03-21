---
layout: post
title: nginx GeoIP 모듈을 이용한 국가코드 식별, 차단, 허용
---

웹기반의 서비스를 운영하다 보면 종종 어느 국가에서 접근했는지에 따라서 다른 처리가 필요할 경우가 발생합니다. 때에 따라서는 특정 국가에서의 접근만을 허용하거나 혹은 불허하는 등의 제한이 필요할 때도 있습니다.

nginx를 이용해 서비스가 구축되어 있다면 geoip module을 사용해 간단하게 접근 국가를 식별하고 그에 따라 다른 처리, 차단 혹은 허용을 설정할 수 있습니다.


## nginx geoip module 설치


우선 설치되어있는 nginx에 geoip module 이 포함되어 있는지 확인합니다.

```bash
$ nginx -V
```

`--with-http_geoip_module` 옵션이 확인되지 않는다면 해당 옵션을 추가하여 다시 설치해야 합니다. 

```bash
$ wget http://nginx.org/download/nginx-1.10.1.tar.gz
$ tar xzf nginx-1.10.1.tar.gz
$ cd nginx-1.10.1
$ ./configure --user=nginx --group=nginx --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_gzip_static_module --with-http_stub_status_module --with-mail --with-mail_ssl_module --with-http_geoip_module --with-pcre
$ make
$ sudo make insetall
```


## geoip 설치

`geoip` 패키지도 설치되어 있는지 확인합니다.

```bash
$ yum install geoip
$ ls /usr/share/GeoIP/
```


## nginx geoip 모듈 설정

nginx 설정에서 geoip 처리를 설정합니다.
모든 국가 접근을 허용하되 한국에서의 접근만 차단하고 싶다면 다음과 같이 설정합니다.

##### /etc/nginx/nginx.conf

```conf
http {
    ...
    # geoip 국가 데이터 파일 위치 지정
    geoip_country /usr/share/GeoIP/GeoIP.dat;

    # geoip 국가 코드 맵핑, 기본 yes, 한국 no
    map $geoip_country_code $allowed_country {
        default yes;
        KR no;
    }

    ...
    server {
        ...
        location / {
            if ($allowed_country = no) {
                return 403;
        }
    ...
```

## ELB 연동 시 IP 전달

만약 ELB - nginx 구조로 nginx를 사용하고 있다면 nginx의 `$remote_addr`가 모두 ELB의 private ip로 잡히게 됩니다.
ELB가 알고있는 실제 클라이언트의 IP는 `X-Forwarded-For`헤더를 통해 전달되게 되므로  `nginx.conf`에 다음과 같은 설정을 추가해 해당 IP주소를 사용하도록 합니다.

```
http {
    real_ip_header X-Forwarded-For;
    set_real_ip_from 10.0.0.0/8;
```

## access log에 국가 코드 출력

nginx의 access.log에 국가 코드가 출력되도록 하고 싶다면 `nginx.conf`에서 `log_format`에 `$geoip_country_code`를 추가합니다.

```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' '$status $body_bytes_sent "$http_referer" ' '"$http_user_agent" "$http_x_forwarded_for" "$realip_remote_addr" "$geoip_country_code"';
```
