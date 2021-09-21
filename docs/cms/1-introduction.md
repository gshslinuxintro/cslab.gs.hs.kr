---
layout: page
title: Contest Management System Docs
subtitle: Introduction
toc: true
#toc_title: Custom Title
menubar: cms_menu
show_sidebar: false
---
# Introduction
CMS (Contest Management System)는 프로그래밍 대회를 구성하는 데 사용하는 오픈소스 소프트웨어입니다. IOI와 같은 대회의 공식 채점 시스템이며, 많은 국가에서 대회를 개최하는 데 사용되고 있습니다. 강력하고, 확장성이 높고, 안전하기 때문입니다.
## General Structure
시스템은 모듈로 구성되어 있기에 각 기능을 서로 다른 서버에서 구동할 수 있습니다. 대회 정보는 ```PostgreSQL```을 이용합니다.
## Services
CMS는 아래와 같은 서비스들을 사용하니다. 모두 다른 서버에서 운영할 수 있습니다.
* ```cmsLogService```: 모든 로그 메시지를 취합해 저장
* ```cmsResourceService```: 서버의 모든 서비스를 한번에 실행
* ```cmsChecker```: 모든 서비스의 상태를 확인인
* ```cmsEvaluationService```: 제출 큐를 관리하고 컴파일, 채점 작업을 ```cmsWorker```에 분배
* ```cmsWorker```: 실제로 작업을 수행행
* ```cmsScoringService```: 채점 결과를 점수로 반영영
* ```cmsProxyService```: 점수를 점수판에 전송
* ```cmsContestWebServer```: 실제 대회가 이루어지는 FE 사이트
* ```cmsRankingWebServer```: 랭킹 사이트
* ```cmsAdminWebServer```: 관리자 사이트








