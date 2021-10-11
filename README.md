
# cslab.gs.hs.kr 개요
경기과학고등학교 레이텍 협회 사이트 저장소입니다. http://cslab.gs.hs.kr/ 에서 만나볼 수 있습니다.

서버에 관한 정보는 [Wiki](https://github.com/gshslinuxintro/cslab.gs.hs.kr/wiki)에서 확인 가능합니다.

## 제공 서비스
* CMS(Contest Management System)
* Status
* Docs (서버, 연구, CMS 설명서)
* Cloud (교내망)

## 설치
이 사이트는 Jekyll을 이용하여 구동됩니다. local 환경에서 운영하려는 경우 다음 과정을 통해 운영할 수 있습니다.
### 저장소 복제
```
git clone https://github.com/gshslinuxintro/cslab.gs.hs.kr
mv cslab.gs.hs.kr /var/www/source
```
### 환경 설치
```
sudo apt-get install ruby-dev gcc make php build-essential nginx-full
```
### nginx 설정
아래 파일에서 포트 번호, hostname 등을 적합하게 수정하시면 됩니다.
```
cp /var/www/source/nginx/cms.conf /etc/nginx/conf.d/cms.conf
```
### Jekyll 설치
```
sudo gem install jekyll bundler
cd /var/www/source
sudo bundle install
```
## 실행
위 작업이 완료되었다면 아래 명령어를 통해 사이트를 실행할 수 있습니다.
이후 http://localhost 나 http://localhost:4000 으로 접속하면 됩니다.
```
sudo bundle exec jekyll serve
```
**평상시에는 그냥 nginx를 이용하세요.**
## 갱신
```update_posts.sh```를 실행하면 수정된 사항들을 갱신할 수 있습니다. (글 갱신 등.)

```/var/www/source``` 안에서 위 실행파일들을 [cron](https://crontab.guru/) 등을 이용해 주기적으로 실행하면 됩니다. 현재 갱신 주기는 각각 1분입니다.


## 글 작성
[CMS](http://cslab.gs.hs.kr/cms/) 에서 글을 작성할 수 있습니다. gshslatexintro에 소속된 github 계정이 필요합니다. [GiHhub Token](https://github.com/settings/tokens)에서 ```repo``` 권한을 가진 ```token```을 생성해 암호란에 입력하면 됩니다. 


## 주의
**경고 : MS 메모장으로는 `.md` 파일을 편집하지 마시오.**
Jekyll 파싱 에러를 야기합니다. 편집할 때에는 GitHub에서 직접 편집하거나, [npp](https://notepad-plus-plus.org/)을 사용하십시오.
It causes Jekyll parsing error. We would recommend to edit and commit on GitHub directly, or edit with [npp](https://notepad-plus-plus.org/).

credit: bulma Jekyll theme
