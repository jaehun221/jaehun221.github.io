---
layout: post
title: rocky-linux install
date: 2024-12-27 09:23:17 +0900
category: linux
---
# rocky-linux 9
> wsl에 rockylinux 설치하기

rocky-linux 공식 가이드: <https://docs.rockylinux.org/guides/interoperability/import_rocky_to_wsl/><br>

위 링크에서 rocky-linux 9 이미지 다운로드

<br>
wsl 설치
```shell
wsl --install
```
<br>
wsl2 환경에 새로운 가상머신 가져오기
```shell
wsl --import <machine-name> <path-to-vm-dir> <path-to/rocky-9-image.tar.xz> --version 2
```
<br>
<span style="background-color: #fff5b1;">wsl --import</span>: WSL에서 새로운 배포판을 가져오는 명령어.<br>

<span style="background-color: #fff5b1;">machine-name</span>: 새로 생성할 WSL 인스턴스(가상 머신)의 이름을 지정.<br>

<span style="background-color: #fff5b1;">path-to-vm-dir</span>: 새로운 WSL 인스턴스의 루트 파일 시스템이 저장될 디렉터리 경로를 지정.<br>  

<span style="background-color: #fff5b1;">path-to/rocky-9-image.tar.xz</span>: 가져올 Rocky Linux 9 이미지 파일의 경로를 지정(tar.xz형식의 압축 파일).

<br>
설치된 배포판 확인
```shell
wsl -l -v
```
<br>
설치된 배포판 실행
```shell
wsl -d <배포판이름>
```
<br>
wsl 배포판 default로 실행
```shell
wsl -d <배포판이름>
```