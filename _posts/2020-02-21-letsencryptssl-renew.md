---
layout: post
title:  "Let's Encrypt SSL 인증서 갱신 오류 해결하기"
date:   2020-02-21 12:00:00
categories: server
tags: ssl dns renew 
comments: true
---

이번에 개인서버를 이전하면서 인증서 파일도 같은경로로 다 가지고 와서 인증서 갱신을 하려고 했더니 아래와 같이 오류가 발생했습니다.

``` bash
certbot-auto renew

Cert is due for renewal, auto-renewing...
Plugins selected: Authenticator standalone, Installer None
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for img.clubheadwear.com
Waiting for verification...
Challenge failed for domain img.clubheadwear.com
http-01 challenge for img.clubheadwear.com
Cleaning up challenges
Attempting to renew cert (img.clubheadwear.com) from /etc/letsencrypt/renewal/img.clubheadwear.com.conf produced an unexpected error: Some challenges have failed.. Skipping.

```

인증서 갱신을 위해서 인증서버쪽에 통신중 오류가 발생하는 것 같은데 방화벽을 열고 여러가지 방법을 찾아봐도 해결이 안되어 마지막을 TXT레코드를 이용한 방법을 사용해보기로 했습니다.

## TXT 레코드 요청

certbot을 실행할때 --manual 옵션과 --preferred-challenges dns를 주어 실행합니다.

```
/data/src/certbot-auto certonly -d img.clubheadwear.com --manual --preferred-challenges dn
```

IP로깅을 허용하겠냐고 묻는 화면에서 Y를 입력합니다.

```
Cert is due for renewal, auto-renewing...
Renewing an existing certificate
Performing the following challenges:
dns-01 challenge for img.clubheadwear.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
```

아래와 같이 DNS 레코드를 변경하라고 메시지가 나옵니다.

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.img.clubheadwear.com with the following value:

31ttHcDECIqEgSx0dwE7ftmfpAb2IejqP_HCAJHFvVY

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue

```

보유한 도메인의 DNS 레코드 설정페이지로 가서 해당 내용을 등록하고 위 Shell 에서 엔터를 친다.
```
_acme-challenge.img.clubheadwear.com	TXT		31ttHcDECIqEgSx0dwE7ftmfpAb2IejqP_HCAJHFvVY
```

그럼 아래와 같이 완료가 된다.

```
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/img.clubheadwear.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/img.clubheadwear.com/privkey.pem
   Your cert will expire on 2020-05-21. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

이제 이후에는 certbot-auto renew 를 입력하여 갱신이 가능하다.

끝