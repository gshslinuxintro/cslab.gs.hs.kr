---
layout: page
title: Contest Management System Docs
subtitle: Installation
toc: true
#toc_title: Custom Title
menubar: cms_menu
show_sidebar: false
---
# Installation
{% include notification.html message="본 설명서는 Ubuntu 20.04 LTS를 기반으로 제작되었다." %}
## Kernel Options
```bash
CONFIG_CGROUPS=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_MEMCG=y
CONFIG_CPUSETS=y
CONFIG_PID_NS=y
CONFIG_IPC_NS=y
CONFIG_NET_NS=y
```

## Dependencies
```bash
sudo apt-get install build-essential openjdk-11-jdk-headless \
     fp-compiler  postgresql postgresql-client cppreference-doc-en-html \
     cgroup-lite libcap-dev zip python3.8 python3.8-dev libpq-dev \
     libcups2-dev libyaml-dev libffi-dev python3-pip nginx-full
```
## Preperation
먼저 Github에서 cms-dev 저장소를 가져온다.
```bash
git clone https://github.com/cms-dev/cms.git --recursive
```
이후 cmsuser 계정과 그룹을 만들기 위해 다음 파이선 파일을 실행한다.
```bash
sudo python3 prerequisites.py install
```
이 스크립트를 실행하면 '이 계정을 cmsuser에 추가하시겠습니까?'라는 질문이 등장한다. 이 계정으로 cms를 운영할 것이라면 y를 입력하면 된다.
만약 나중에 cmsuser 계정을 추가하고 싶다면, 다음 명령어를 통해 추가할 수 있다.
```bash
sudo usermod -aG cmsuser <user_name>
```
## Installing CMS and its Python dependencies
먼저 가상환경을 만든다.
```bash
python3 -m venv ~/cms_venv
```
이후 이 가상환경을 활성화한다.
```bash
source ~/cms_venv/bin/activate
```
이 환경 내에서 cms-dev를 설치한다.
```bash
pip3 install -r requirements.txt
python3 setup.py install
```
아래 명령어를 통해 가상환경을 해제한다.
```bash
deactivate
```
{% include notification.html message="이후 CMS를 이용할 때는 항상 가상환경을 이용해야 한다." status="is-danger" %}
