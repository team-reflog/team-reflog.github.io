---
layout: post
title: |
    Route 53으로 GitHub pages 서브 도메인 연결하기
date: 2024-01-14T04:28:00.000Z
categories: [Infrastructure]
tags: [AWS, GitHub]
---


## 시작하며


👋 안녕하세요! 이번 글에서는 Route 53을 이용하여 구입한 도메인의 서브 도메인으로 GitHub pages의 DNS 주소를 변경해보겠습니다.


이 글은 다음 자료를 참고하여 작성하였습니다.

- [Managing a custom domain for your GitHub Pages site - GitHub Docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

## 알아두기


우선 본격적으로 시작하기 전에 **반드시** 알아야할 사항이 몇 가지 있습니다.


### 비용


AWS Route 53의 사용할 때 청구되는 요금은 다음과 같습니다.

- 호스팅 영역: 한 달에 각 $0.5
- 표준 쿼리: 한 달에 100만 개의 쿼리당 $0.4

이 글에서는 호스팅 영역 1개를 생성하고 100만 개 미만의 쿼리를 가정하므로 **한 달에 $0.9의 요금이 청구**됩니다. 자세한 내용은 아래를 참고해주세요.

- [Amazon Route 53 요금 - Amazon Web Services](https://aws.amazon.com/ko/route53/pricing/)

### 준비물


시작하기 전, 다음 항목들이 준비되어 있어야 합니다.

- 배포된 GitHub pages 리포지토리
- 도메인 구매

이번 글에서는 [Reflog의 팀 블로그 리포지토리](https://github.com/team-reflog/team-reflog.github.io)와 [가비아](https://www.gabia.com/)에서 구매한 도메인으로 진행해보겠습니다.


## Route 53 호스팅 영역 생성


우선 AWS로 이동하여 로그인 후 검색 창에 아래와 같이 검색하여 Route 53 서비스로 이동합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/c65b4626-5434-4eb4-98c3-c02aa7c613ee/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=cd65fccc9fb96941c5e74fc323040130fc8e2dbc4a69a77cc081ec767fa9f261&X-Amz-SignedHeaders=host&x-id=GetObject)


좌측 메뉴에서 `Hosted zone`을 클릭합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/8e094d52-24fe-4648-b8b9-8b147009a0a7/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=d331638d72bf3673c06b0be48f3b55435b93d0bccff36f52944098a2df3bf78a&X-Amz-SignedHeaders=host&x-id=GetObject)


`Create hosted zone`을 눌러 호스팅 영역 생성 화면으로 이동합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/a406066b-6f25-4360-afb2-a2e9095f9ddc/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=b16ebd37c404a6c720e5258c7de11b9964bc7fcdf3ef3ca0a7ee32f74bd21762&X-Amz-SignedHeaders=host&x-id=GetObject)


`Domain name`에 구입한 도메인을 입력한 후 `Create hosted zone`을 눌러 호스팅 영역을 생성합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/19d2eaff-9ba5-4d31-84fc-90ed6c8ea726/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=7ce52a9e1d86b5b6ae218eae85674589c0d2030fbacdafec91b108bfc402276f&X-Amz-SignedHeaders=host&x-id=GetObject)


다음과 같이 기본적으로 두 레코드가 생성됩니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/da35085f-5eb6-4a1d-a5b9-d8117e34d71e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=d5cd5e8032e36a0e5a33c6972e535e47b4706ccba3289dff1b95c45296faf95a&X-Amz-SignedHeaders=host&x-id=GetObject)


여기서 NS 레코드의 4개의 값을 도메인 서비스 사이트에서 입력하여 네임 서버를 변경해야합니다. 가비아의 경우 아래 링크를 참고해주세요.

- [네임서버 관리하기 - 가비아 고객센터](https://customer.gabia.com/manual/domain/286/991)

> NS 레코드는 "Name Server" 레코드의 약어로, 도메인 이름 시스템(Domain Name System, DNS)에서 사용됩니다. NS 레코드는 특정 도메인에 대한 네임 서버 정보를 지정합니다. 이 레코드를 통해 해당 도메인의 DNS 서버가 어디에 위치해 있는지 식별할 수 있습니다. 네임 서버는 도메인 이름을 IP 주소로 해석하거나, 다른 DNS 정보를 제공하는 역할을 합니다.


> 네임 서버는 DNS에서 사용되는 서버로, 인터넷 상에서 도메인 이름과 IP 주소를 연결시키는 역할을 합니다. 사용자가 웹 브라우저에서 도메인 이름을 입력하면, 네임 서버는 해당 도메인의 IP 주소를 찾아서 사용자 컴퓨터에 전달합니다.


## 서브 도메인 연결하기


이제 보유한 도메인으로부터 서브 도메인을 GitHub pages에 연결할 수 있습니다. 이를 위해선 CNAME 레코드를 생성해야합니다.


> CNAME 레코드는 "Canonical Name"을 나타냅니다. 이는 도메인 이름 시스템(DNS)에서 사용되는 레코드 유형 중 하나로, 도메인을 다른 도메인에 별칭(alias)으로 매핑할 때 사용됩니다. CNAME 레코드를 사용하여 한 도메인의 DNS 레코드를 다른 도메인의 레코드로 연결할 수 있습니다. 이를 통해 특정 도메인에 대한 요청이 다른 도메인으로 전달되어, 두 도메인이 동일한 IP 주소를 공유하거나 서비스를 공유할 수 있습니다.


### CNAME 레코드 생성하기


생성한 호스팅 영역 화면에서 `Create record`를 눌러 레코드 생성 창으로 이동합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/0ef77064-8e8e-4da6-9c1a-13aa414a0aad/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=f6252a808ccd46b70d39bdd2af48828e4b987d292811323a659bc268c85eaaf6&X-Amz-SignedHeaders=host&x-id=GetObject)


원하는 대로 값을 채워 넣습니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/e62eff08-a937-4b48-bac9-83c4da51de5b/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=2e51ac7060605ed92560484c772adee8751d74aea61e2dea8303cbddceaf25a3&X-Amz-SignedHeaders=host&x-id=GetObject)


생성한 CNAME이 생성되었는지 확인합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/16cad60e-dc35-4e3b-bea1-484f8acaef04/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=d1af62b97a308197c5b7e7627cc307d332bad02ebe91bc760c0f2e0ad2db635b&X-Amz-SignedHeaders=host&x-id=GetObject)


### CNAME 동작 확인


새로 생성한 레코드가 정상적으로 GitHub pages에 연결되었는지 확인하기 위해 다음 명령어를 쉘에 입력합니다.


```sql
dig YOUR_RECORD_NAME +nostats +nocomments +nocmd
```


예를 들어 이번 글의 예시의 경우 다음과 같이 입력하여 결과를 확인할 수 있습니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/8eb4f89e-6af9-4433-8fae-482b54b5c82a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=7e4072e5a064f5374bb5571001aefdd37e0290079610c4626a8816321d171507&X-Amz-SignedHeaders=host&x-id=GetObject)


## GitHub pages 서브 도메인 등록하기


이제 거의 다 끝났습니다. GitHub pages 리포지토리에서 `Setting` > `Pages`으로 이동합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/4a480733-f21f-4abf-b0f2-034118a90f6e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=de93b640932d8f5e574a5c08d3ed8672a6bead1f290df3f7354b427c67578595&X-Amz-SignedHeaders=host&x-id=GetObject)


`Custom domain` 항목에 생성한 도메인을 입력한 후 `Save`를 눌러 저장합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/8315df7c-a754-4765-8cbb-8e9a1bc2069f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=3b2400edfdf3239496c77841dc3dea4fa68c0f41b8b8c447a2f39413fcbbc31b&X-Amz-SignedHeaders=host&x-id=GetObject)


DNS 확인 프로세스가 실행되고 잠시 후 연결이 정상적으로 완료됩니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/8dabbd1e-2fde-483c-a702-e9e3098f4be4/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T180800Z&X-Amz-Expires=3600&X-Amz-Signature=6ded26c42762781e3fd795b8f4451fc6081f295e73a3fd9fbdbd449c6b4886a6&X-Amz-SignedHeaders=host&x-id=GetObject)


### 확인하기


본인의 커스텀 도메인으로 이동하여 정상적으로 GitHub pages가 표시되는지 확인합니다.


## 마무리하며


이번 글에서는 구입한 도메인을 Route 53에서 등록하여 호스팅 영역을 생성하였습니다. 이후 CNAME 레코드를 추가하여 커스텀 도메인을 GitHub pages와 연결하였습니다.


GitHub pages과 구입한 도메인을 이용하여 정적 웹 페이지를 만들고 싶은 분들에게 도움이 되었길 바랍니다. 감사합니다.

