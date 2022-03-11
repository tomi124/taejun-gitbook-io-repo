---
description: SSL 인증서 검증에 대해서 알아보자
---

# SSL

## DCV cname 쿼리 조회

&#x20;nslookup -q=cname 레코드

\


## 인증서 체크

openssl verify -CAfile 도메인.pem 도메인.crt

openssl verify -verbose -CAfile 도메인.pem 도메.crt

openssl verify  도메인.pem

\


## 인증서 만료일 체크

openssl x509 -in 도메 -noout -enddate |cut -c10-40

\


## 인증서 비밀번호 체크

open rsa -in 도메.key -modulus -noout | openssl md5

openssl x509 in 도메.crt -modulus -noout | openssl md5

\


\


## 인증서 검증

\


openssl rsa -noout -text -in 도메.key -modulus -passin pass:패스워| grep Modulus

openssl x509 -noout -text -in 도메인.crt -modulus | grep 'Modulus\\='

위 2개의 값 비교!



\


## 실서버 체크

\


openssl s\_client -CApath 도메인.pem -connect 도메인:443
