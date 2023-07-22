## 도커 , 쿠버네티스 실습
### 도커 실습
- [도커사용법 기본](./dockeredu/docker1.md)
- [켄테이너 외부 접속](./dockeredu/docker2.md)
- [이미지 생성 ,저장소 push,도커파일](./dockeredu/docker3.md)
### 쿠버네티스 실습
- [파드,디플로이먼트,서비스,레플리카](./k8sedu/podservice/pod.md)
- [인그레스](./k8sedu/ingress/ingress.md)

### 사전 준비사항
- Docker Desktop 설치 https://docs.docker.com/desktop/mac/install/ .

  
### Docker 실습 - 기본 
- ngix-pod.yaml작성
```
docker -v
Docker version 24.0.2, build cb74dfc
```

```
mkdir dockerEdu
cd dockerEdu
```
- 유분투 컨테이너를 생성 및 실행를 동시에 하고 컨테이너 내부로 들어가보자. 
```
docker run -i -t ubuntu:14.04
Unable to find image 'ubuntu:14.04' locally
14.04: Pulling from library/ubuntu
2e6e20c8e2e6: Pull complete
0551a797c01d: Pull complete
512123a864da: Pull complete
Digest: sha256:64483f3496c1373bfd55348e88694d1c4d0c9b660dee6bfef5e12f43b9933b30
Status: Downloaded newer image for ubuntu:14.04
root@e6bba9a67da7:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@e6bba9a67da7:/# exit
exit
```
- 도커이미지 저장소에서 centos 내려받기 
```
docker pull centos:7
7: Pulling from library/centos
2d473b07cdd5: Pull complete
Digest: sha256:be65f488b7764ad3638f236b7b515b3678369a5124c47b8d32916d6487418ea4
Status: Downloaded newer image for centos:7
docker.io/library/centos:7

What's Next?
  View summary of image vulnerabilities and recommendations → docker scout quickview centos:7
```
- 도커 엔진에 존재하는 이미지 목록 출력
```
docker images
```
- 컨테이너 생성을 위한 create 명령어 사용
```
docker create -i -t --name mycentos centos:7
```
- start 명령어와 attach 명령어를 써서 컨테이너를 시작하고 내부로 들어가 보자.
```
 docker start mycentos
mycentos
 docker attach mycentos
[root@61bcfcc52556 /]# ls
anaconda-post.log  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@61bcfcc52556 /]#
```
- Ctrl + p를 사용해서 나오자. 
- 현재까지 생성한 컨테이너 확인
```
docker ps
```
- ps 는 정지되지 않은 컨테이너만 보여줌 , 정지된 컨테이너를 포함한 모든 컨테이너 출력하려면 -a옵션 추가
```
docker ps -a
```
- 컨테이너 삭제 rm
```
docker rm mycentos
```
- 혹시 실행 중이면 정지한 뒤 삭제해야 함 . 또는 rm 에 -f 옵션으로 정지후 삭제 가능 
```
docker stop mycentos
docker rm mycentos
```
- 모든 컨테이너 삭제 명령어
```
docker contaniner prune
```