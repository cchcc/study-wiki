Install
---
[https://about.gitlab.com/downloads/](https://about.gitlab.com/downloads/)

설정은 `/etc/gitlab/gitlab.rb` 여기서

```
external_url 'http://your.domain:port'  // 포트 or url 설정
```
재설정

```
sudo gitlab-ctl reconfigure
```
시작/종료/재시작

```
sudo gitlab-ctl start
sudo gitlab-ctl stop
sudo gitlab-ctl restart
```

최초 접속시 root 유저 비번 변경설정
