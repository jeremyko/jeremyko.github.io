---
layout: post
title: nerdtree euc-kr 환경에서 사용하기
date: '2018-09-27T23:00:00.001+09:00'
tags:
    - euc-kr
    - vim
    - NERDTree
modified_time: '2021-04-03T16:00:01.390+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-1650378270143398402
blogger_orig_url: https://jeremyko.blogspot.com/2018/09/nerdtree-euc-kr.html
---

<h3> <span style="color:{{site.span_h3_color}}"> 
nerdtree euc-kr 환경에서 utf8 환경인 것처럼 사용하기 위한 꼼수
</span> </h3>

항상 vim + nerdtree 를 조합해서 작업하고 있고 nerdtree없이 작업하는것은 매우 끔찍한 시간이 될거 같다. 그런데 최근 새로 일하게 된 곳의 서버는 euc-kr 로 설정이 된 상태이다 (LC_CTYPE=ko_KR.eucKR). 그리고 계정은 1개뿐이라서, 개발자 별 로 설정을 달리 할수도 없는 상황이다.  
<span style="color:{{site.span_emphasis_color}}"> 이 환경에서는 nerdtree 플러그인이 제대로 동작하지 않는다 (utf8 에서 정상동작)</span>

<h3> <span style="color:{{site.span_h3_color}}"> 
1. 자신의 터미널 세션만 변경하고 ~/.vimrc 를 수정하는 방법
</span> </h3>

열려 있는 터미널에서만 환경변수를 다음을 실행하여 변경하고,

    export LANG=ko_KR.utf8
    export LC_CTYPE=ko_KR.utf8

`~/.vimrc` 에 다음처럼 설정해 주고,

    set encoding=utf-8

이런 식으로 하면 `~/.vim` 에 nerdtree 설치해서 나는 사용할 수는 있으나, 이것은 vi 사용자 모두에게 영향을 주고, 다른 사용자로 부터 욕을 먹는 경우가 발생한다.
nerdtree 관련된 플러그인 파일들이 존재하면 vim 를 실행하는 시점부터 타 사용자들은 nerdtree 플러그인 로딩 실패 관련된 에러가 발생하기 때문이다.

<h3> <span style="color:{{site.span_h3_color}}"> 
2.  vim 기동시 나만의 설정을 가지고 동작되게 한다
</span> </h3>

그래서 방법을 찾다가 다음처럼 사용 중인데 나름 괜찮은 듯하다.

vim 기동시 `-u` 옵션을 주면 임의 위치의 vim 설정 파일을 지정 할수 있다.  
즉 기본값 `~/.vimrc` 말고 다른 설정 파일을 지정할수 있다.

또한 임의 지정한 vim 설정 파일에서 임의 위치의 .vim 디렉토리 위치를 지정할수 있다.  
<span style="color:{{site.span_emphasis_color}}">즉 `~/.vim` 디렉토리 대신 임의 위치를 지정 가능하다.</span>

자 그럼, `-u` 옵션을 사용하기 위해, 임의 위치에 설정파일을 하나 만든다. 예를 들어서 custom_vimrc 이름으로 `~/users/kojh/` 디렉토리에 파일을 생성하고, 내용은 다음처럼 한다.

    """ 자신이 사용중인 .vimrc 내용을 설정하면 된다.한가지 주의점은
    """ set nocompatible 가 제일 먼저, 꼭 설정 되어야 함.
    set nocompatible
    """let g:NERDTreeDirArrows=0
    """let NERDTreeDirArrowExpandable ='+'
    """let NERDTreeDirArrowCollapsible ='~'
    let NERDTreeIgnore = ['\.pyc$','\.o$']
    map :nt <ESC>:NERDTree<CR>
    set sw=4
    set sts=4
    set ts=4
    set autoindent
    set expandtab
    set smartindent
    set ruler
    set nu
    set hls
    set rulerformat=%70(%<%40.45f\ %m\ %r%=%l/%L,\ %cpp%V\ %4P%)
    syntax on
    colorscheme desert
    set colorcolumn=80,130
    """ 기타 등등 ......

    set encoding=utf-8
    set fileencodings=utf-8,euc-kr

    """ 다음 부분이 핵심이다.
    let &runtimepath='~/.vim_kojh,$VIMRUNTIME'
    let $VIMHOME='~/.vim_kojh'

<span style="color:{{site.span_emphasis_color}}">let &runtimepath ... </span>이부분이 중요한데, 위에 설명한것처럼 euc-kr 환경에서는 `~/.vim` 에 nerdtree를 설치 할수 없기때문에 `~/.vim_kojh` 라는 임의 폴더를 생성해서 그 안에 nerdtree 플러그 인을 설치한다. 그리고 그 경로를 지정해 준것이다.

이런 작업을 통해서 기존의 `.vimrc`, `.vim` 디렉토리와 완전 별개의 설정으로 vim 을 구동시킬수 있다.

자신의 터미널 에뮬 설정을 utf8로 설정하는것도 반드시 필요 (즉, putty 등 설정에서 encoding 부분)

한가지 문제점은 만약 vundle 과 같은 plug-in 매니저를 사용중인 경우에는  
vundle 내부적으로 `~/.vim` 위치를 하드코딩 한 듯하며, 이 때문에 문제가 발생한다.

이때는 그냥

-   `~/users/kojh/custom_vimrc` 에서 vundle 부분을 제거하고(--;)
-   `~/.vim_kojh/bundle/nerdtree` 내의 모든 파일을 `~/.vim_kojh/` 위치로 옮기면 된다.  
    (즉, `~/.vim_kojh` 위치에서 mv bundle/nerdtree/\* . 실행).

vundle 과 같은 plug-in 매니저를 사용하는 방법은 불가능해보이지만.. 이건 아마 다른 방법이 있을지도 모르겠다.

set nocompatible 설정은 다른 방법도 있다. vim -N 옵션을 사용하면 된다.

<h4> <span style="color:{{site.span_h4_color}}"> 
이렇게 설정이 완료되면 vim 실행은 다음처럼 수행한다
</span> </h4>

-   먼저 해당 쉘의 환경변수를 재설정한다(현재 세션만 영향을 받게)

    export LANG=ko_KR.utf8
    export LC_CTYPE=ko_KR.utf8

-   그리고 vim -u ~/users/kojh/custom_vimrc 로 옵션을 주고 실행한다.

좀더 간단하게 한번에 하려면 alias 를 설정하면 된다.

    alias kovim='. ~/users/kojh/myprofile;vim -u ~/users/kojh/custom_vimrc '

    이경우 ~/users/kojh/myprofile 파일을 생성하고 그내용은 위의 export 2줄을 넣으면 된다.

utf-8 환경에서 그냥 cat 명령으로 euc-kr 문서를 보면 깨지므로 다음처럼 alias를 만들어서 사용해도 좋다.

    alias cat-euc-utf8='iconv -f EUC-KR -t UTF8 '

좀 번잡스럽기는 하지만 이런식으로 euc-kr 환경에서 nerdtree를 utf8 환경에서 나만 독립적으로 사용할수 있다. 물론 제일 좋은 해결책은 서버 설정을 utf8 로 맞추는 것이다. 아마 소스코드를 윈도우에서 편집 하거나, DB 에 한글 저장한다고 이런 설정을 오래전부터 유지하고 있는 듯 한데, utf8로 한글 처리 안되는것 도 아니니 왠만하면 설정을 utf8 로 하는게 맞다고 본다.. 아무튼 시대에 뒤떨어진 설정을 가진 서버들도 있으니 이런 방법이 유용할 경우도 있겠다.
