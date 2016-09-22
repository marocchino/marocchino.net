---
title: vi-mode
date: 2016-09-23 03:57:49
tags:
- vim
- bash
- shell
---

# tl;dr

RTFM

# 동기

처음에는 이맥스 단축키 외우기 귀찮고 빔이 더 편하지 않을까 정도로 쓰기
시작했다만, 한 2년간 사용했는데도 인터프리터랄께 셸마다 미묘하게 동작이 틀리기도
하고 아예 사용할 수 없는 경우도 있어서 그냥 이맥스 모드도 익숙해져버렸다.

익히고 까먹고 익히고 까먹고를 반복하고 있어서 정리할 겸..

# 설정

기본적으로는 `.inputrc`에 추가 하기만 하면 된다.

```bash
set editing-mode vi
```

다만, `fzf`를 사용한다면 [오동작](https://github.com/junegunn/fzf/issues/39)이
생기니 `.bashrc`에서 로드하기 전에 한번 더 설정해줘야 한다.

```bash
set -o vi
[ -f ~/.fzf.bash ] && source ~/.fzf.bash
```

이것만해도 사용하는데는 크게 문제 없는데, 가끔 지금 어떤 모드에 있는지 몰라서
삽질할 때가 있으니 프롬프트에 현재 상태를 표시하게 해뒀다.
`.inputrc`에 추가로..

```bash
set show-mode-in-prompt on
set vi-ins-mode-string \1\e[34;1m\2✚ \1\e[0m\2
set vi-cmd-mode-string \1\e[35;1m\2: \1\e[0m\2
```

# 사용하기

전채 목록은 [readline](https://linux.die.net/man/3/readline)의 Default Key
Bindings 항목을 읽어보면 된다. 이정도는 알아 둘만 한 것들만 뽑아봤다.

| 명령 | 이맥스 | 빔 |
|------|--------|----|
| 맨 앞으로 이동 | ctrl-a | esc I |
| 맨 뒤로 이동 | ctrl-e | esc A |
| 단어단위 앞으로 이동 | ctrl-b | esc b |
| 단어단위 뒤로 이동 | ctrl-f | esc w |
| 커서위치부터 뒤 삭제 | ctrl-k | esc D |
| 에디터에서 열기 | ctrl-x ctrl-e | esc v |
| 히스토리 검색 | ctrl-r | ctrl-r |
| 주석 처리 | alt-# | esc I# |
| 클리어 | ctrl-l | esc ctrl-l |

# 문제점

아까도 말했는데 몇몇 콘솔은 vi mode를 무시한다. 대표적으로
[Erlang](http://stackoverflow.com/questions/11976765/erlang-interpreter-vi-mode)하고
그 위에서 돌아가는 [iex](https://github.com/elixir-lang/elixir/issues/4533)가
그렇다. 물론 래퍼로 감싸서 [우회해서 사용](https://gist.github.com/jfreeze/8894279)할
수는 있지만, 리스크 관리는 알아서;

vi mode는 있는데 명령을 사용할 수 없는 경우도 있다. fish에서는 `esc v`가
동작하지 않는다.

# 참고

- https://linux.die.net/man/3/readline
- https://github.com/jlevy/the-art-of-command-line/blob/master/README-ko.md

