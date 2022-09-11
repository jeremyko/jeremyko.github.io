---
layout: post
title: Python Flask 설치 (windows)
date: '2012-09-10T13:06:00.000+09:00'
tags:
    - python
    - flask
modified_time: '2012-09-10T13:06:33.490+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-937013726216875892
blogger_orig_url: https://jeremyko.blogspot.com/2012/09/python-flask-windows.html
---

### 당연히 python은 먼저 설치되어 있어야 함.

### easy_install, pip 설치

이것들은 Python 패키지들을 설치하고 관리하기 위한 도구이다. 다음 순서대로 수행.

-   easy_install 설치
    [http://python-distribute.org/distribute_setup.py](http://python-distribute.org/distribute_setup.py) 을 다운받아서 실행.

-   PC path에 `C:\Python27\Scripts` 를 추가.

-   pip 설치 : `easy_install pip`

### 이제 Flask를 설치 할수 있다. 다음을 실행.

    pip install Flask

### 여기서 부터는 옵션.

**virtualenv**

서로 다른 버전의 파이선, 라이브러리들과 작업해야 하는 경우에 자신의 프로젝트만을 위한 파이선 개발 환경을 만들수 있다. 사용하고 싶다면 다음을 실행.

    pip install virtualenv
    (혹은 easy_install virtualenv 으로도 가능)

그리고 프로젝트 폴더상에서 다음을 실행

    virtualenv venv

이제 관련 폴더들이 생성되고, 해당 프로젝트를 위한 환경이 구성되었다.
해당 환경을 활성화 시키기 위해 다음을 실행한다.

    venv\scripts\activate

프롬프트가 (venv) 로 변경됨을 알아두자. 이상태에서, `pip install Flask` 을 수행해서 개발환경을 만들수 있다.

이경우, `\venv\Lib\site-packages` 에 관련 패키지가 설치된다(즉, 프로젝트별 환경이 구성됨).
