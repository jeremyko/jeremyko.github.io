---
layout: post
title: 맥미니  2009 late  ubuntu , nvidia 9400M driver 설치하기
date: '2019-08-16T19:28:00.007+09:00'
tags:
    - ubuntu
modified_time: '2021-04-03T15:57:58.548+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3088279466382990696
blogger_orig_url: https://jeremyko.blogspot.com/2019/08/2009-late-ubuntu-nvidia-9400m-driver.html
---

<h4> <span style="color:{{site.span_h4_color}}"> 
2021 update :  
NVIDIA 드라이버 제 성능이 안나와서 그냥 구석에 쳐박혀서 사용 안함. 결국 정말 쓸데가 없는 물건임.
</span> </h4>

ubuntu 18.04 에서 nvidia driver 잘못깔린 경우, 부팅이 안되는 상황에서 대처 방안 및 제대로된 driver 설치하기.

집에 오래된 mac mini 2009 late 모델이 있는데 이젠 더이상 os upgrade도 안되고 그러다보니 apple music 불가. firefox 도 제대로 사용이 안되서 별로 해볼수 있는게 없다. 그래서 ubuntu 전용 머신으로 변신 시켜보고자 시도해봤다. usb 로 시동 디스크를 만들어서 부팅까지 성공은 했는데, nvidia driver를 잘못 깔아서 인지 부팅이 안되는 상황이 발생했고, 이런것 처음이다보니 여기저기 기웃거리면서 삽질한것을 정리해본다. .

먼저 ubuntu로 안전모드로 부팅한다. 부팅시에 esc 키를 !한번만! 눌러준다.
메뉴에서 "Drop to root shell prompt” 를 선택, shell 로 들어간다.
잘못 설치된 nvidia driver를 제거해줘야 한다. 다음 명령을 차례대로 수행한다.

    mount -n -o remount, rw /
    apt-get purge nvidia-\*
    reboot

이제 부팅은 정상적으로 진행된다.
이제 기본적으로 설치된 nouveau 라고 하는 드라이버를 삭제해준다.
아래 경로에 blacklist 파일을 생성한다.

    $ sudo vi /etc/modprobe.d/blacklist-nouveau.conf

생성된 .conf 파일에 아래 두 줄을 입력한다.

    blacklist nouveau
    options nouveau modset=0

아래 명령어를 입력 후 재부팅한다.

    $ sudo update-initramfs -u
    $ sudo reboot
    nvidia driver를 설치한다.
    $ sudo add-apt-repository ppa:graphics-drivers/ppa
    $ sudo apt update
    $ sudo ubuntu-drivers autoinstall
    $ sudo reboot

정상적으로 driver가 설치됬는지 다음 명령으로 확인한다.

    nvidia-smi

<h3> <span style="color:{{site.span_h3_color}}"> 
참고한 사이트 
</span> </h3>

[https://driz2le.tistory.com/254](https://driz2le.tistory.com/254)  
[http://sarghis.com/blog/1043/](http://sarghis.com/blog/1043/)
