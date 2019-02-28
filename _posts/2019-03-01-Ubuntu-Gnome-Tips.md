---
layout: post
title: "Ubuntu Gnome Tips"
date: 2019-03-01 01:00:00 +0900
category: linux, ubuntu, gnome, tech, tip
---

## 설정을 /usr 에서 고치지 말 것

`/usr`은 프로그램들이 주로 설치되는 곳이고 apt를 통해 업데이트 하거나 할 때마다 갈아치워지는 곳이다.

가끔 리눅스 팁이라고 `/usr/share` 에 있는 설정파일을 고치라는 것을 올리는 사람들이 있는데 이는 보통 관습을 잘 모르는 초보자가 쓴 잘못된 팁이다. 그렇게 하면 apt를 통해 패키지가 업그레이드 될 때마다 설정을 열심히 찾아서 고쳐야 한다.

대체로 머신 전체에 적용하고 싶은 설정은 `/etc`에, 사용자에만 적용하고 싶은 설정은 `$HOME` 혹은 `$HOME/.config`에 저장한다.

## 한영키와 한자키가 ralt, rctrl로 먹힐 때

`localectl set-x11-keymap`을 사용한다.

형식
```
localectl set-x11-keymap LAYOUT [MODEL [VARIANT [OPTIONS]]]
```

예시
```
localectl set-x11-keymap kr pc105 qwerty korean:ralt_rctrl
```

model이나 variant는 모르면 `""`로 놔둬도 무방하다. 중요한 것은 마지막에 적절한 options가 지정되는 것이다.  
(다만 대부분의 경우 model은 pc105거나 pc104일 것이다.)

예시2
```
localectl set-x11-keymap kr "" "" korean:ralt_rctrl
```

키보드에 따라서 한영키와 한자키가 ralt, rctrl도 아닌 다른 키로 매핑되어 있을 수도 있는데 이 때에는 적절한 옵션을 찾거나 일일히 매핑해야 한다.

`korean:ralt_rctrl`과 같은 한국어 키보드에 유용한 옵션 목록은 `/usr/share/X11/xkb/symbols/kr`에서 찾을 수 있다.

## 기본 앱 설정

https://specifications.freedesktop.org/mime-apps-spec/latest/ar01s04.html 참조

`.config/mimeapps.list`에서 기본앱을 설정할 수 있다.

Arch Gnome에서 vs code를 깔고 디렉토리에 `xdg-open`을 하면 탐색기(Nautilus) 대신 vs code가 열려버려서 알게 된 것.

```
[Default Applications]
inode/directory=org.gnome.Nautilus.desktop;
```

`/usr/share/applications/mimeinfo.cache`를 고치란 소리가 나와있는 곳도 있는데 헛소리이다. 첫 항목 참조.

## apt 저장소와 gpg 키 관리

apt에서 repository를 추가하려면 주소만 필요한 것이 아니라 그곳에서 받아온 패키지들의 진위여부를 확인할 수 있는 gpg key도 추가되어야 한다.

repository의 주소는 `/etc/apt/source.list`와 `/etc/apt/source.list.d/*.list`에 저장된다.

gpg key 정보는 `/etc/apt/trusted.gpg`와 `/etc/trusted.gpg.d/*.gpg`에 저장된다.

그런데 docker 설치를 해보면 자동 스크립트가 냅다 `/etc/apt/source.list`와 `/etc/apt/trusted.gpg`에 각각 자신의 repository 주소와 gpg key를 추가해버리는 것을 볼 수가 있다.

반면 vs code를 설치해보면 `/etc/apt/source.list.d/vscode.list`와 `/etc/apt/trusted.gpg.d/microsoft.gpg`를 생성하여 처리한다.

나는 vs code처럼 별개의 파일로 저장하고 더 이상 해당 repository를 원하지 않을 때에 쉽게 지울 수 있는 방식을 선호하기 때문에 굳이 docker도 별개의 파일에 저장하도록 나누었다.

`source.list`는 사람이 읽을 수 있게 되어 있기 때문에 `/etc/apt/source.list.d/docker.list`를 만들어 옮기는 데에 큰 어려움은 없다.

다만 gpg key 파일은 읽어서 옮길 수 있게 생기지 않아서 `apt-key` 명령을 이용한다.

1. docker key id 파악  
  `sudo apt-key list` 로 현재 apt에서 가지고 있는 키 목록을 받을 수 있다.  
  4글자씩 늘어서 있는 줄이 keyid이고 마지막 8글자만 써도 keyid로 사용할 수 있다.
  ```
  /etc/apt/trusted.gpg
  --------------------
  pub   rsa4096 2017-02-22 [SCEA]
        9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
  uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
  sub   rsa4096 2017-02-22 [S]
  ```
  위와 같은 경우 docker의 gpg key id는 0EBFCD88 이다.  
2. gpg key export  
  해당 키를 export하여 적당한 곳에 저장한다.
  ```
  sudo apt-key export 0EBFCD88 > ~/docker.pub
  ```
3. delete gpg key  
  원래 있던 키를 지운다
  ```
  sudo apt-key del 0EBFCD88
  ```
  잘 지워졌는지 `sudo apt-key list`로 확인한다.
4. reimport gpg key  
  `--keyring` 옵션을 사용해서 어떤 파일에 해당 gpg key를 기록할 것인지를 지정하여 `add`한다.
  ```
  sudo apt-key --keyring /etc/apt/trusted.gpg.d/docker.gpg add ~/docker.pub
  ```
  역시 잘 추가 되었는지 `sudo apt-key list`로 확인한다.
  ```
  /etc/apt/trusted.gpg.d/docker.gpg
  ---------------------------------
  pub   rsa4096 2017-02-22 [SCEA]
        9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
  uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
  sub   rsa4096 2017-02-22 [S]
  ```
5. 정리  
  잘 되었으면 중간에 저장한 gpg key 파일은 삭제한다.
  ```
  rm ~/docker.pub
  ```
