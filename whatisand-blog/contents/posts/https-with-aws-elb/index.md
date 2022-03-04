---
title: "AWS EC2에 배포된 서버 HTTPS 통신 적용하기"
description: "EC2에 Docker로 배포된 API서버에 외부에서 구매한 도메인을 이용하여 SSL인증서를 적용하며 배운 것을 정리합니다."
date: 2022-02-01
update: 2022-02-01
tags:
  - AWS
  - EC2
  - SSL
  - til
  - 작성중
series: "오늘의 개발일기"
---

> 이 글은 지속적으로 작성되며 업데이트 되는 글입니다. 지금은 별다른 내용이 없을 수 있는 점 양해 부탁드립니다.

## 배경

사이드 프로젝트로 개발중인 API서버를 슬슬 배포해보고자 합니다. 프론트앤드와 백앤드를 따로, 다른시간에 개발하다 보니 항상 띄워져 있을 개발 서버의 필요성을 느끼게 되었습니다. 이왕 하는김에 도메인까지 구매해서 개발서버로 사용해보면 어떨까 생각이 들었습니다.

더욱 이왕 하는 김에 SSL까지 적용하여 실제 서비스 환경과 동일하게 개발을 진행하려고 합니다.
방법을 조금 찾아보니 대부분의 문서에서 AWS 내부에서 도메인 구매부터 모든 것을 진행하는 방법이 소개되고 있었습니다. 저희는 도메인은 AWS가 아닌 다른 곳에서 구매하였는데 이 과정에서 진행한 경험을 공유해봅니다.

- 외부에서 구매한 도메인을 보유함
- 서버에서 SSL을 직접 적용하고 싶지는 않음


## 적용 과정
### 도메인 구매 & Cloudfront로 관리하도록 세팅


### AWS 네임서버 등록

### AWS 인증서 발급 신청

### 타겟 그룹 설정
여기서 영역 설정을 잘못 해서 한참 고생함.

### Application Loadbalncer 세팅
여기서 인증서 적용함


## 배운 점