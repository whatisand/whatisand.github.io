---
title: "Github Actions를 이용해 Docker CD 도전기"
description: "깃허브 액션으로 Fast API Docker 빌드부터 배포까지 진행해봅니다."
date: 2022-02-04
update: 2022-02-04
tags:
  - ec2
  - github
  - CD
  - Docker
  - til
  - 작성중
series: "오늘의 개발일기"
---


> 이 글은 지속적으로 작성되며 업데이트 되는 글입니다. 지금은 별다른 내용이 없을 수 있는 점 양해 부탁드립니다.

## 배경
Fast API를 프로젝트에 무작정 써보기 위해 Fast API 개발자님이 만들어 주신 보일러플레이트를 이용했습니다.
구조를 단순히 따라가는 것으로도 공부가 되는 기분이 들었습니다. 다만 처음부터 해봐야 공부가 되는 것들도 이미 세팅이 되어있었기 때문에 따로 공부를 해야 한다는 생각을 하고 있습니다.

배포와 관련해서 Docker를 써보고 싶었는데, 이미 제가 사용한 보일러플레이트에는 실행 가능하도록 dockerfile과 docker-compose 파일이 세팅 되어있었습니다. 덕분에 처음부터 개발 환경 세팅에 시간을 쓸 것 없이 편하게 비즈니스 로직 개발을 진행할 수 있었습니다.

배포에 즈음하여 고민했습니다. 도커를 쓰고 있지만 아직 배포시에는 도커의 순기능을 활용하지 못하고 있었습니다. 다른 팀에서는 개발자분들이 이미 세팅해주셨기 때문에 Github에 merge만 해도 배포가 되었었는데 그것이 참 그리웠습니다.

도커라이징도 되어있겠다 Github Actions를 이용하여 CD를 구성해보며 이 글을 작성합니다.


## 접근 방법
우선 Github Actions는 어깨너머로 본 경험이 있었기에 기본적인 구조는 알고 있었습니다. 
깃허브 도커 이미지 EC2 배포하기와 관련해 리서치를 통해 다음과 같은 방식으로 진행된다고 이해했습니다.



**배포 액션 계획**
1. develop에 merge 또는 push 된 경우
2. 코드 체크 스크립트 실행 (Type check, Testing)
3. 도커 이미지 파일 빌드
4. 깃허브 패키지에 업로드
5. EC2에 뭔가 명령어 실행


### 막힌 점
1. 깃헙에서 도커 이미지를 빌드해서 특정 저장소에 등록하는 것 까지는 이해함. 근데 ec2에서 해당 도커 파일을 받는 것은 어떤 액션을 통해 진행되는 걸까?
	1. Github actions에 runnners라고 하는 무언가가 있음.
	2. 우리가 주로 쓰는 방식은 Github에서 자체 호스팅 해주는 runner에서 빌드하는 방식임
	3. self-hosted runner 방식을 이용해서 내 ec2에서 코드가 실행될 수 있도록 하는 방식으로 진행하면 됨


## 참고자료
- [Vue 프로젝트 Github Action & Docker 이용해서 EC2에 배포하기 -- 1](https://velog.io/@zlemzlem5656/Vue-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-Github-Action-Docker-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-EC2%EC%97%90-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0)
- [Github Actions으로 AWS EC2에 CI/CD 구축하기](https://velog.io/@soosungp33/Github-Actions%EC%9C%BC%EB%A1%9C-AWS-EC2%EC%97%90-CICD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0)
- [Github Actions Self-hosted 공식문서](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)