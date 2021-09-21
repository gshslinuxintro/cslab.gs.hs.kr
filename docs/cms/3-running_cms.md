---
layout: page
title: Running CMS
subtitle: Installation
toc: true
#toc_title: Custom Title
menubar: cms_menu
show_sidebar: false
---
# Running CMS
## Configuring the DB
다음 명령어를 통해 PostgreSQL 사용자에 로그인한다.
```bash
sudo su - postgres
```
이후, ```cmsdb```와 그 관리자를 만든다.
{% include notification.html message="한번에 붙여넣지 말고, 한줄씩 각각 붙여넣는다. 중간에 role의 비밀번호를 입력하는 단계가 있음에 유의하라." status="is-danger"  %}
```bash
createuser --username=postgres --pwprompt cmsuser
createdb --username=postgres --owner=cmsuser cmsdb
psql --username=postgres --dbname=cmsdb --command='ALTER SCHEMA public OWNER TO cmsuser'
psql --username=postgres --dbname=cmsdb --command='GRANT SELECT ON pg_largeobject TO cmsuser'
```
이후 ```cms.conf``` 파일을 구성해야 한다. 다음 챕터를 먼저 확인하라.

cms.conf를 정확하게 구성했다면 다음 명령어를 통해 데이터베이스를 시작하면 된다.
```bash
cmsInitDB
```
{% include notification.html 
message="만약 여러 개의 서버에서 서비스를 운영한다면, 다음과 같이 PostgreSQL의 접속을 허가해야 한다. ```postgresql.conf```에 다음과 같이 LISTENING ADDRESS를 추가한다.
```bash
listen_address = '127.0.0.1, xxx.xxx.xxx.xxx'
```
또, HBA가 요청을 허가하도록 다음을 ```pg_hba.conf```에 추가한다.
```bash
host    cmsdb   cmsuser 192.168.0.0/24 md5
```
"  status="is-info"%}
## Configuring CMS
CMS는 2개의 config 파일을 이용한다. ```/usr/local/etc/```에 위치한 ```cms.conf```와 ```cms.ranking.conf```다.
* ```cms.conf```는 cms의 여러 서비스들이 이용하는 포트, db를 비롯한 모든 설정이 담겨 있는 파일이다.
* ```cms.ranking.conf```는 ```cmsRankingWebServer```와 관련된 설정을 다룬다.
이중 ```cms.conf```에서는 다음을 수정해야 된다.
* db 접속 문자열(cmsuser, 아까 PostgrSQL의 비밀번호)
* secret_key(초깃값을 바꿀 것)
* ranking 통신 계정(실제 리눅스 계정 X)

여기서 port를 8888, 8889, 8890 대신 다른 값으로 바꿀 수 있다.
## Building CMS
주요한 수정사항이 발생했을 경우, 이전 설치 과정에서 실행했던 명령을 반복하면 된다.
```bash
prerequisites.py install
```
## Running CMS
아래 명령어를 통해 관리자를 추가한다.
```bash
cmsAddAdmin -p <password> <username>
```
이후 관리자 페이지를 시작한다.
```bash
cmsAdminWebServer
```
여기서 Creat New Contest를 통해 테스트 콘테스트를 만든다. 

앞으로 ```cmsResourceService -a```를 입력하면 cws, aws 등을 모두 한번에 작동시킬 수 있다.