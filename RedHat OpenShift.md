github access token
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

# 1장

- 레지스트리 : 컨테이너 이미지 저장 공간(저장소)
- 레포지토리 : 이미지를 관리하는 단위.

- hub.docker.com : 도커 컨테이너 이미지 레지스트리
- image ID : ID로 변경안됨.
- repository : 변경 가능.

```
podman pull httpd
	- httpd 도커 이미지 다운로드.

podman images
	- 저장된 도커 이미지 확인.
	- 계정(사용자)별 이미지 저장소 존재로 공유되지 않음.
		- root의 이미지와 student의 이미지가 서로 따로관리됨.

podman run [ubi8/ubi:8.3] [echo 'Hello, world!']
  - 도커 이미지 실행
  - 두번째 인자를 주면, 컨테이너에 기본 입력된 명령어 대신 두번째 인자의 명령어 실행
  - 본 경우, Hello world를 출력
```
