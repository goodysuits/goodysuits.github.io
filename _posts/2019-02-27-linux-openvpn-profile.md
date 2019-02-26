---
title: ubuntu에서 config파일로 openvpn client 실행&서비스 등록하기
key: 20190227
tags: linux ubuntu network openvpn
---

<!--more-->

이 포스트는 아래의 환경에서 작성된 포스트입니다.

* systemd를 사용하는 linux

대게 SSL/TLS 인증 방식을 사용하는 VPN서버에 접속하기 위해서 openvpn을 클라이언트로 사용한다.
인증을 위해서는 아래의 파일들이 준비되어 있을 것이다.

* ca.crt
* client.key
* client.crt
* client.conf

conf 파일이 준비되어 있지 않다면 아래 파일을 참조해서 conf 파일을 작성하자.

```
/user/share/doc/openvpn/examples/sample-config-files/client.conf
```

위 4개의 파일을 /etc/openvpn 으로 옮긴 후,

```
$ sudo openvpn --config client.conf
```

위 명령어로 conf파일이 적용된 openvpn client를 실행할 수 있다.

서비스로 이용하기 위해서는

```
$ sudo service openvpn@client start
```

이 명령어로 openvpn client를 서비스로 실행할 수 있다.

linux를 재시작한 후에 자동으로 service를 실행하기 위해서는

```
$ sudo systemctl enable openvpn@client
```

위 명령어를 실행하면 된다.
