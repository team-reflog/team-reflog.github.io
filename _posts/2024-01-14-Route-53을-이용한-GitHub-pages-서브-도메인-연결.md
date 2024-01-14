---
layout: post
title: |
    Route 53을 이용한 GitHub pages 서브 도메인 연결
date: 2024-01-14T04:28:00.000Z
categories: [Infrastructure]
tags: [AWS, GitHub]
---


## 시작하며


👋 안녕하세요! 이번 글에서는 Route 53을 이용하여 구입한 도메인의 서브 도메인으로 GitHub pages의 DNS 주소를 변경해보겠습니다.


이 글은 다음 자료를 참고하여 작성하였습니다.

- [Managing a custom domain for your GitHub Pages site | GitHub Docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

## 알아두기


우선 본격적으로 시작하기 전에 **반드시** 알아야할 사항이 몇 가지 있습니다.


### 비용


AWS Route 53의 사용할 때 청구되는 요금은 다음과 같습니다.

- 호스팅 영역: 한 달에 각 $0.5
- 표준 쿼리: 한 달에 100만 개의 쿼리당 $0.4

이 글에서는 호스팅 영역 1개를 생성하고 100만 개 미만의 쿼리를 가정하므로 **한 달에 $0.9의 요금이 청구**됩니다. 자세한 내용은 아래를 참고해주세요.

- [Amazon Route 53 요금 | Amazon Web Services](https://aws.amazon.com/ko/route53/pricing/)

### 준비물


시작하기 전, 다음 항목들이 준비되어 있어야 합니다.

- 배포된 GitHub pages 리포지토리
- 도메인 구매

이번 글에서는 [Reflog의 팀 블로그 리포지토리](https://github.com/team-reflog/team-reflog.github.io)와 [가비아](https://www.gabia.com/)에서 구매한 도메인으로 진행해보겠습니다.


## Route 53 호스팅 영역 생성


우선 AWS로 이동하여 로그인 후 검색 창에 아래와 같이 검색하여 Route 53 서비스로 이동합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/c65b4626-5434-4eb4-98c3-c02aa7c613ee/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=a443dcb20c5ddf0894f7c25c51e10f2258ee776ee77328f9adea2ff692a77842&X-Amz-SignedHeaders=host&x-id=GetObject)


좌측 메뉴에서 `Hosted zone`을 클릭합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/8e094d52-24fe-4648-b8b9-8b147009a0a7/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=bdc95fd8fa1a32fc82a8386706bec916c4b4d06c8752d8a76919e50a8ec041ad&X-Amz-SignedHeaders=host&x-id=GetObject)


`Create hosted zone`을 눌러 호스팅 영역 생성 화면으로 이동합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/a406066b-6f25-4360-afb2-a2e9095f9ddc/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=6345d97885af21fa71b834dea9e85ad34eb9f4c259bb3992a2d2e1919e9f80cf&X-Amz-SignedHeaders=host&x-id=GetObject)


`Domain name`에 구입한 도메인을 입력한 후 `Create hosted zone`을 눌러 호스팅 영역을 생성합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/19d2eaff-9ba5-4d31-84fc-90ed6c8ea726/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=6563073c9c94c81855f135e9f5755960c749738d8a07a2b7ec0d456dab30f87f&X-Amz-SignedHeaders=host&x-id=GetObject)


다음과 같이 기본적으로 두 레코드가 생성됩니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/da35085f-5eb6-4a1d-a5b9-d8117e34d71e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=e58d43b5a5c50c66a1f33d1647e2f0945c78f581237e6359960586d689dd3263&X-Amz-SignedHeaders=host&x-id=GetObject)


여기서 NS 레코드의 4개의 값을 도메인 서비스 사이트에서 입력하여 네임 서버를 변경해야합니다. 가비아의 경우 아래 링크를 참고해주세요.

- [네임서버 관리하기 | 가비아 고객센터](https://customer.gabia.com/manual/domain/286/991)

> NS 레코드는 "Name Server" 레코드의 약어로, 도메인 이름 시스템(Domain Name System, DNS)에서 사용됩니다. NS 레코드는 특정 도메인에 대한 네임 서버 정보를 지정합니다. 이 레코드를 통해 해당 도메인의 DNS 서버가 어디에 위치해 있는지 식별할 수 있습니다. 네임 서버는 도메인 이름을 IP 주소로 해석하거나, 다른 DNS 정보를 제공하는 역할을 합니다.


> 네임 서버는 DNS에서 사용되는 서버로, 인터넷 상에서 도메인 이름과 IP 주소를 연결시키는 역할을 합니다. 사용자가 웹 브라우저에서 도메인 이름을 입력하면, 네임 서버는 해당 도메인의 IP 주소를 찾아서 사용자 컴퓨터에 전달합니다.


## 서브 도메인 연결하기


이제 보유한 도메인으로부터 서브 도메인을 GitHub pages에 연결할 수 있습니다. 이를 위해선 CNAME 레코드를 생성해야합니다.


> CNAME 레코드는 "Canonical Name"을 나타냅니다. 이는 도메인 이름 시스템(DNS)에서 사용되는 레코드 유형 중 하나로, 도메인을 다른 도메인에 별칭(alias)으로 매핑할 때 사용됩니다. CNAME 레코드를 사용하여 한 도메인의 DNS 레코드를 다른 도메인의 레코드로 연결할 수 있습니다. 이를 통해 특정 도메인에 대한 요청이 다른 도메인으로 전달되어, 두 도메인이 동일한 IP 주소를 공유하거나 서비스를 공유할 수 있습니다.


### CNAME 레코드 생성하기


생성한 호스팅 영역 화면에서 `Create record`를 눌러 레코드 생성 창으로 이동합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/0ef77064-8e8e-4da6-9c1a-13aa414a0aad/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=4d11ba2a0b411ef4bf493a1c87178fecf8d893fcac815cc6696bcc27f619be24&X-Amz-SignedHeaders=host&x-id=GetObject)


원하는 대로 값을 채워 넣습니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/e62eff08-a937-4b48-bac9-83c4da51de5b/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=f2d3cfbafdca5f750a14b420970d6ee81671c88d5c2f1581444dade93bee9e93&X-Amz-SignedHeaders=host&x-id=GetObject)


생성한 CNAME이 생성되었는지 확인합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/16cad60e-dc35-4e3b-bea1-484f8acaef04/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=7910b1f441fe1522b8def6969fb03bd3b7a2fcc238d0bbfdc5b12a40b64b9266&X-Amz-SignedHeaders=host&x-id=GetObject)


### CNAME 동작 확인


새로 생성한 레코드가 정상적으로 GitHub pages에 연결되었는지 확인하기 위해 다음 명령어를 쉘에 입력합니다.


```sql
dig YOUR_RECORD_NAME +nostats +nocomments +nocmd
```


예를 들어 이번 글의 예시의 경우 다음과 같이 입력하여 결과를 확인할 수 있습니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/8eb4f89e-6af9-4433-8fae-482b54b5c82a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=a80f1b0ad5d2084a9ad91ede9fb323431a388f305bfffc7fa84f2a3aa2ce958e&X-Amz-SignedHeaders=host&x-id=GetObject)


## GitHub pages 서브 도메인 등록하기


이제 거의 다 끝났습니다. GitHub pages 리포지토리에서 `Setting` > `Pages`으로 이동합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/4a480733-f21f-4abf-b0f2-034118a90f6e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=dac4041fc56595bfdc702a863083f1690de9dc0aeb85b403c68210da077e8f45&X-Amz-SignedHeaders=host&x-id=GetObject)


`Custom domain` 항목에 생성한 도메인을 입력한 후 `Save`를 눌러 저장합니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/8315df7c-a754-4765-8cbb-8e9a1bc2069f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=a28bcdd0998c224b7855377c4cf5430504c84fa6e253fb4d1f01685055bfef9e&X-Amz-SignedHeaders=host&x-id=GetObject)


DNS 확인 프로세스가 실행되고 잠시 후 연결이 정상적으로 완료됩니다.


![Untitled.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/59f25ad3-0b68-4f51-aa09-d7d59b3488b5/8dabbd1e-2fde-483c-a702-e9e3098f4be4/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240114T064709Z&X-Amz-Expires=3600&X-Amz-Signature=c6af9de295d469805062267a0167e0c285586cf581c9f8a57f23f290f52115a4&X-Amz-SignedHeaders=host&x-id=GetObject)


### 확인하기


본인의 커스텀 도메인으로 이동하여 정상적으로 GitHub pages가 표시되는지 확인합니다.


## 마무리하며


이번 글에서는 구입한 도메인을 Route 53에서 등록하여 호스팅 영역을 생성하였습니다. 이후 CNAME 레코드를 추가하여 커스텀 도메인을 GitHub pages와 연결하였습니다.


이번 글이 GitHub pages과 구입한 도메인을 이용하여 정적 웹 페이지를 만들고 싶은 분들에게 도움이 되었길 바랍니다. 감사합니다.

