---
title: "[Security] Certificate file format & extensions
date: 2021-08-27 15:57:28 -0400
categories: security
tags:
  - security
comments: true
---

## Encoding (확장자로 쓰이기도 함.)
- `.der`
	- Distinguished Encoding Representation (DER)
	- binary DER 형식으로 encoding 된 인증서
- `.pem`: X.509 v3 파일의 한 형태
	- PEM (Privacy Enhanced Mail)은 Base64인코딩된 ASCII text file이다.
	- 원래는 secure email에 사용되는 인코딩 포멧이었는데 더이상 email쪽에서는 잘 쓰이지 않고 인증서 또는 키값을 저장하는데 많이 사용
	-    `-----BEGIN XXX-----`,  `-----END XXX-----`  로 묶여있는 text file을 보면 이 형식으로 인코딩 되어있다고 생각하면 된다. (담고있는 내용이 무엇인지에 따라 XXX 위치에 CERTIFICATE, RSA PRIVATE KEY 등의 키워드가 들어있다)
	- 인증서(Certificate = public key), 비밀키(private key), 인증서 발급 요청을 위해 생성하는 CSR (certificate signing request) 등을 저장하는데 사용된다.

## 확장자
- `.crt`, `.cer` 인증서를 나타내는 확장자인 cer과 crt는 거의 동일하다고 생각하면 된다. (cer은 Microsoft 제품군에서 많이 사용되고, crt는 unix, linux 계열에서 많이 사용된다.)
-  `.key`: 개인 또는 공개 PKCS#8 키를 의미
- `.p12`: PKCS#12 형식으로 하나 또는 그이상의 certificate(public)과 그에 대응하는 private key를 포함하고 있는 key store 파일이며 패스워드로 암호화 되어있다. 열어서 내용을 확인하려면 패스워드가 필요하다.
- `.pfx`: PKCS#12는 Microsoft의 PFX파일을 계승하여 만들어진 포멧이라 pfx와 p12를 구분없이 동일하게 사용하기도 한다

## 표준 비교 PKCS#8, PKCS #12, X.509
- PKCS #8
	- Public-Key Cryptography Standards (PKCS) 표준 중의 일부로 private key를 저장하는 문법에 관한 표준
	- PKCS #8 private keys 는 일반적으로 PEM 형식으로 인코딩된다.
- PKCS #12
	- 하나의 파일에 여러 암호화 관련 엔티티 들을 합쳐서 보관하는 방식에 관한 표준
- X.509
	- public key infrastructure (PKI)에 대한 ITU-T의 표준 (RFC 5280)
	- 공개키(public key) 인증서의 format, revocation list(더이상 유효하지 않은 인증서들에 대한 정보 배포), certification path(chain) validation 알고리즘 등을 정의
	- 보통 X.509 인증서라 하면 RFC 5280에 따라서 인코딩되거나 서명된 디지털 문서를 말함.


> 출처
> - https://www.letmecompile.com/certificate-file-format-extensions-comparison/
