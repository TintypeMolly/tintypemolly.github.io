---
layout: post
title: "Arch Linux, GNOME, Plymouth 설정하기"
date: 2018-08-15 03:30:00 +0900
categories: linux
tags: linux arch gnome plymouth
---

내 랩탑은 OS 미탑재 모델이고, 싸구려 랩탑에 그다지 돈을 쓰고 싶지 않았기 때문에 윈도우를 사지 않고 Arch Linux를 설치하여 사용하고 있다.

Windows 10을 깔았으면 아예 만나지 않았거나 금방 해결되었을 문제들이 지속적으로 발생하는데, 지금 생각나는 것만 써 보겠다.

* 무선 인터넷에 접속하는 법을 잘 몰라서 네트워크 관리 서비스 2개를 설치해 충돌하게 해놓고 왜 안 되는지 헤메기
* 그래픽 성능이 너무 낮아서 유튜브 영상도 제대로 못 보는 현상이  xf86-video-intel이 없어서 그런 건줄 알았다가 gdm이 Xorg가 아니라 Wayland를 사용해서였다는 것을 깨닫기
* 전원이 연결되어 있을 때와 배터리 전력을 사용할 때 각각  suspend와 hibernate를 하게 하고 싶은데 GUI 설정 프로그램에서는 안 보여서 gsetting을 어떻게 조작하는지 한참을 찾아보기
* plymouth를 빌드하면서 glibc가 'sys/sysmacros.h' include를 없애버린 것 때문에 빌드오류가 나서 고생하기
* yay(AUR Package Manager의 일종)으로 PKGBUILD를 변조해서 설치하는 법 몰라서 찾느라 고생하기

기억나지 않는 것까지 합하면 더 많을 것이다.

각설하고, 하여간 그런 손 많이 가는 linux지만 원하는대로 설정 가능한 OS라는 건 프로그래머(혹은 컴퓨터 오타쿠)에게는 굉장히 매력적인 일이다.
linux라고 하면 보통 켜지자마자 CUI로 로그인 Prompt가 뜨고 로그인해서 ls나 cd 따위의 명령어를 사용하고 emacs나 vim 같은 무시무시한 에디터를 사용해서 코딩하는 이상한 사람들만 쓰는 OS라고 생각하는 사람이 많을 것이다.
하지만 GNOME이나 Xfce같은 Desktop Environment를 사용하면꽤 데스크탑과 비슷하게 사용할 수 있다.
나는 GNOME을 설치했고, gdm도 사용하고 있어서 처음에 grub의 메시지나 udev같은 모듈들의 로딩시 로그들을 제외하면 Windows나 macOS와 딱 보기에는 별로 차이가 없다(라고 약을 팔고 싶다).
거기서 더 나아가 그 처음 로딩시의 무서운 텍스트들도 없애면 기분이 좋을 것 같아서 조사를 하여 `Plymouth`라는 것이 있다는 것을 알게 되었다.

홈페이지: https://www.freedesktop.org/wiki/Software/Plymouth/

Gnome에서 Plymouth의 설치 과정은 대략 이렇다.

1. AUR에서 받아와서 plymouth와 gdm-plymouth를 설치
  yay나 pacaur같은 AUR 패키지 관리 도우미를 사용한다면 gdm-plymouth만 설치하면 의존성으로 plymouth를 설치할 것이다.
  이 때, 2018년 08월 13일 현재 glibc의 버전업에 의해 plymouth가 깨져있기 때문에 해당 레포의 PKGBUILD를 좀 수정해줘야 한다.
  https://627690.bugs.gentoo.org/attachment.cgi?id=488690 이 링크에 존재하는 패치를 적당한 곳에 위치시킨 뒤,
  PKGBUILD의 `prepare()` 함수의 마지막 줄에 그 패치를 적용시키는 내용을 추가시키면 된다.
  예를 들어 위 패치를 `/home/foo/sysmacros.patch`에 위치시켰다고 한다면
  `patch -p1 -i /home/foo/sysmacros.path`를 추가해주면 되는 식이다.
  내가 사용하는 yay의 경우 `--editmenu` 옵션을 인자로 넣어주면 어떤 패키지의 PKGBUILD를 고칠 거냐고 물어본다.
  `yay --editmenu -S plymouth` 와 같은 식으로 사용하면 된다.
  기존에 설치된 gdm이 있다면 gdm-plymouth를 설치하면서 충돌이 날 수 있는데, 그냥 기존의 gdm을 지우고 gdm-plymouth를 사용하면 된다.
  그리고 `sudo systemctl disable gdm.service' 한 다음 `sudo systemctl enable gdm-plymouth.service'를 해주면 된다.
2. initramfs에 plymouth hook 추가하기
  initramfs HOOKS는 `base udev ...` 같은 식으로 나올텐데, `udev` 뒤에 `plymouth`를 넣어주면 된다.
  나는 initramfs를 빌드하는 데에 mkinitcpio를 사용하고 있고, `/etc/mkinitcpio.conf` 에 `HOOKS`을 고쳐주면 된다.
  그렇게 하면 `udev` hook이 실행되는 것만 보인 뒤 바로 `plymouth`에서 지정한 테마의 로딩 화면이 보이는 설정이 된다.
  또한 `MODULES`에 적절한 그래픽 모듈을 지정해줘야 그림이 적절하게 표시될 수 있다.
  나는 내장 Intel 그래픽을 사용 중이었기 때문에 `MODULES`에 `i915`를 지정해주면 되었는데, 다른 회사의 그래픽카드는 그래픽 모듈 이름이 다르다는 것 같다.
  설정 변경을 완료했으면 루트 권한으로 `mkinitcpio -p linux` 해서 initramfs를 빌드하면 된다.
3. 커널 파라메터 설정하기
  나는 grub을 사용하기에 `/etc/default/grub.cfg`에서 `GRUB_CMDLINE_LINUX_DEFAULT` 값을 수정해줬다.
  `splash` 인자가 있어야 동작하는 것 같은데 달리 문서에 씌어져 있는 것이 없어서 확실치 않다.
  하여간 나는 커널 파라메터로 `quite`와 `splash`를 설정해 두었다.
  설정파일을 변경하고 나서 루트 권한으로 `grub-mkconfig -o /boot/grub/grub.cfg` 해줘야 반영된다.

이렇게 하면 grub 화면이 지나가고 나서 udev 로그가 보인 뒤 부팅되게 된다. 스플래시 화면의 생김새는 원하는 plymouth 테마를 적용하면 된다.
https://www.gnome-look.org/ 라는 웹사이트에서 'Arch Breeze'라는 테마를 적용했는데 상당히 멋진 것같다.
`yay -S plymouth-theme-arch-breeze` 로 설치한 뒤 `sudo plymouth-set-default-theme -R arch-breeze` 를 통해 간단히 적용할 수 있었다.

그리고 남은 grub과 udev 메시지에 대해서인데, 이 둘도 덜 무섭게 보이게 할 수 있다.

https://www.gnome-look.org/ 에는 grub 테마도 있어서 grub을 덜 무섭게 보이게 하는 것은 상당히 간단했다.
그냥 저 홈페이지에서 GUI-ish한 테마를 찾아서 `/etc/default/grub`에서 `GRUB_THEME`으로 지정해주면 끝나는 일이다.
`Arch silence`라는 이름의 grub 테마를 적용했고 꽤 GUI적으로 보여서 만족했다.
또 별로 도움되지 않는 메시지도 삭제해버렸는데, `Loading Linux ...` 라든가 `Loading initial ramdisk ...` 같은 메시지가 지저분해 보여서 삭제해버렸다.
이런 메시지는 `/etc/grub.d/10_linux` 같은 파일 안에 있는데, `Loading Linux` 같은 것을 텍스트 찾기로 찾아서 그것을 `echo` 하는 곳을 주석처리 해버리면 된다.
설정을 변경하고 나면 까먹지 말고 `sudo grub-mkconfig -o /boot/grub/grub.cfg` 해줘야 한다.

그리고 마지막으로 남은 관문은 grub에서 initramfs로 주도권이 넘어간 뒤, base와 udev 훅이 plymouth 훅보다 먼저 불리기 때문에 생기는 텍스트 표시이다.
udev는 `starting version 200` 같은 메시지를 찍는데 이게 스플래시 화면 나오기 전에 나와서 꽤 거슬린다.
이것은 다시 한 번 커널 파라메터를 건드리면 되는데, `rd.udev.log-priority=3`를 커널 파라메터에 추가하면 표시되지 않는다.
이건 우연히 발견한 방법이고 어떤 의미인지 나는 확실히 알고있지 못한데, 아마도 `KERN_ERR` 이상의 긴급도가 아니면 표시하지 않는다는 의미인 것 같다.

이렇게 하면 처음 켜지면서
1. grub이 예쁘게 표시되고
2. initramfs에 주도권이 넘어가면 base와 udev는 조용히 빈 화면에 깜빡이는 텍스트 커서만 보이다가
3. plymouth가 스플래시 화면을 표시하고
4. 모든 부팅이 완료되면 gdm의 로그인 화면이 반긴다.

이렇게 부팅 과정에서 생 텍스트가 거의 표시되지 않으므로 방구석에서 체제전복적인 사이버 테러를 계획하는 너드 의혹을 조금은 해소할 수 있을 것이다.
