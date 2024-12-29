Docker tip
---

```sh
https://gist.github.com/nacyot/8366310

도커 허브에서 이미지 가져오기
docker pull ubuntu:14.04

도커 이미지를 컨테이너로 실행 (bash로 컨테이너 안에 들어감)
이미지가 없으면 자동으로 받아옴
docker run -it ubuntu:14.04 /bin/bash

-i                           Keep STDIN open even if not attached
-t                           Allocate a pseudo-TTY
-p                           Publish a container's port(s) to the host (default [])

도커 이미지를 컨테이너로 실행
docker start (이름 혹은 id)

도커 컨테이너에 들어가기
docker attach (이름 혹은 id)


도커 컨테이너 안에 들어가있는 상태에서 배시쉘만 빠져나오기
ctrl + p, q

도커 컨테이너 정지
docker stop (이름 혹은 id)

도커 컨테이너 삭제
docker rm (이름 혹은 id)

도커 이미지 삭제
docker rmi (이름 혹은 id)


https://github.com/jupyter/docker-stacks
docker run -it -p 8888:8888 jupyter/tensorflow-notebook
```