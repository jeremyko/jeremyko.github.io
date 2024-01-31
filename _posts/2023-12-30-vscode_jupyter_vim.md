---
layout: post
title: 'vscode, jupyterlab 에서 vim 사용시 esc keymap 문제 해결'
date: 2023-12-30 12:01:01 +0900
# categories: python # 별도 폴더로 관리된다 !!!
tags: [jupyter,vim,python]
---


'esc'를 눌러  insert mode 종료 시 현재 cell 밖으로 나가버리는 문제를 해결하기 위한 깨알 tip.


#### vscode keybindings.json 에 다음 추가

즉, vim 사용 중(normal 모드)이라면 esc 무시 

    [
        {
            "key": "escape",
            "command": "",
            "when": "vim.active && vim.mode == 'Normal'"
        },
    ]


#### 만약 , jupyterlab 을 사용하는 경우

jupyterlab-vim을 설치하고

    pip install jupyterlab-vim

메뉴 settings --> setting editor --> Notebook Vim으로 들어가서 다음 항목의 체크를 해제한다.

    Enable `Esc` and `Ctrl-[` leaving vim Normal mode to Jupyter Command mode 




### 참고

[https://github.com/VSCodeVim/Vim/issues/5238#issuecomment-719990357]( https://github.com/VSCodeVim/Vim/issues/5238#issuecomment-719990357)

[https://github.com/jupyterlab-contrib/jupyterlab-vim](https://github.com/jupyterlab-contrib/jupyterlab-vim)





