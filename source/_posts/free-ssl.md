---
title: Free SSL
date: 2018-05-23 16:51:24
tags: [OpenSSL]
cover: lets-encrypt.png
categories: [tip]
subtitle: 무료로 SSL 사용하기
---
# 수동으로 SSL 적용하기
HTTPS 환경에서 개발 연동이 필요하여 임시로 Let`s Encrypt로 무료 SSL을 사용함.
이 글은 Let`s Encrypt을 사용할 때마다 헤깔려하는 나를 위함.
나중에 기회가 되면 Let`s Encrypt 자동 갱신 방법도 올릴 예정임.

※ 참고 사이트
https://blog.outsider.ne.kr/1178
https://www.heydari.be/spring-boot-application-secured-by-a-lets-encrypt-certificate/

### 1. letsencrypt 클라이언트 설치하기 (in 리눅스 환경 – CentOS 7)

#### 1) letsencrypt 클라이언트 clone
``` bash
$ git clone https://github.com/letsencrypt/letsencrypt

```

#### 2) letsencrypt 의존성 다운로드
``` bash
$ ./letsencrypt-auto –help
```

### 2. letsencrypt 인증서 발급

#### 1) 수동으로 인증서 발급
``` bash
$ ./letsencrypt-auto certonly –manual
```

#### 2) 도메인 이름 입력
ex) arcjjang.tk
[참고] freenom으로 무료로 도메인 받아서 설정한 화면
{% asset_img freenom.png freenom %}

#### 3) 마지막으로 동의(y)하면 다음과 같은 안내 메시지와 함께 대기 상태에 들어감
``` bash
create a file containing just this data:

ssQikD07N-XPEQRb-i7Cevxu7pvbt5hrN6hhV02EFiQ.bzLFg1U0rxJirH-2T6iWLLTVs4ptpChlpMm1sekcEDU

And make it available on your web server at this URL:

http://arcjjang.tk/.well-known/acme-challenge/ssQikD07N-XPEQRb-i7Cevxu7pvbt5hrN6hhV02EFiQ

-------------------------------------------------------------------------------
Press Enter to Continue
```

#### 4) 해당 정보로 인증을 받을 수 있는 임시 web 서버를 만들자
##### 4-1) Spring Initializr로 web 선택
##### 4-2) resources 폴더에 위 data를 입력
{% asset_img spring-boot.png spring-boot %}
##### 4-2) application 실행 후 press enter
##### 4-4) 인증 수행 후 다음과 같이 안내메시지가 나오면 성공
``` bash
Press Enter to Continue
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/arcjjang.tk/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/arcjjang.tk/privkey.pem
   Your cert will expire on 2018-08-18. To obtain a new or tweaked
   version of this certificate in the future, simply run
   letsencrypt-auto again. To non-interactively renew *all* of your
   certificates, run "letsencrypt-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

### 3. 생성된 PEM 파일로부터 PKCS12 파일을 생성하자.

#### 1) PEM 파일 위치로 이동
``` bash
$ cd /etc/letsencrypt/live/arcjjang.tk/
```

#### 2) PEM 파일 위치로 이동
``` bash
$ openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out keystore.p12 -name tomcat -CAfile chain.pem -caname root
```

#### 3) keystore.p12 파일 추출

### 4. Spring Boot 어플리케이션 설정하기

#### 1) keystore.p12을 import한다.
```
ex) resources/ssl/keystore.p12
```

#### 2) application.properties 설정하기
``` java
server.port=443
security.require-ssl=true
server.ssl.key-store=classpath:static/ssl/keystore.p12
server.ssl.key-store-password=password
server.ssl.keyStoreType=PKCS12
server.ssl.keyAlias=tomcat
```

### 5. Spring Boot 어플리케이션 실행 후 테스트 화면
{% asset_img result.png result %}