---
title: Bash tips
date: 2016-09-29 15:13:26
tags:
- bash
- fish
---

안적어두면 자꾸까먹어서 정리할 겸..

# 여러 파일 치환

이런 작업은 vim에서 하는거보다는 밖에서 하는게 빠르고 정규식도 POSIX라 편하다.
오타 수정할 때 쓰면 좋다.

```bash
perl -pi -e 's/old-string/new-string/g' my-files-*.txt
```

조금 덧붙여서 커밋된 파일만 순회할 수도 있다.

```bash
perl -pi -e 's/old-string/new-string/g' $(git ls-tree HEAD --name-only -r)
```

특정 폴더만 지정하려면 -- 쓰면된다.

```bash
perl -pi -e 's/old-string/new-string/g' \
  $(git ls-tree HEAD --name-only -r -- ./folder)
```

# clipboard 내용으로 사전열기

번역하고 있을때 유용하게 쓴다.
OSX한정이고 bash tip이 아닐지도 모르겠는데 이렇게 한다.
alias로 해두면 세손가락 더블탭보다 스트로크가 적다.

```bash
open -a /Applications/Dictionary.app/ --args $(pbpaste)
```

# 패턴에 따라 여러파일 지정하기

디렉토리 구조가 비슷하다거나. 확장자만 조금 다르거나 할 때 사용할 수있다.

```bash
vim {ko,.}/lessions/basic/test.md
vim readme.{ko,en}.md
```

# if statement

기본적으로 이렇다.

```bash
#!/bin/bash
if [[ condition ]]; then
  something
else
  something
fi
```

구문은 딱히 특별할거 없고, condition만 신경쓰면 된다.

| 명령 | 설명 |
|------|------|
| ! 표현식 | 표현식이 참이 아님 |
| -n 문자열 | 문자열의 길이가 0보다 큼 |
| -z 문자열 | 문자열의 길이가 0 |
| 문자열1 = 문자열2 | 두문자열이 같음 |
| 문자열1 != 문자열2 | 두문자열이 다름 |
| 숫자1 -eq 숫자2 | 두숫자가 같음 |
| 숫자1 -ne 숫자2 | 두숫자가 다름 |
| 숫자1 -gt 숫자2 | 숫자1이 숫자2보다 큼 |
| 숫자1 -lt 숫자2 | 숫자1이 숫자2보다 작음 |
| -d 파일 | 디렉토리가 있음 |
| -e 파일 | 파일이 있음 |
| -r 파일 | 파일이 있고 읽기권한이 있음 |
| -s 파일 | 파일이 있고 크기가 0보다 큼 |
| -w 파일 | 파일이 있고 쓰기권한이 있음 |
| -x 파일 | 파일이 있고 실행권한이 있음 |

# $변수들

| 표현 | 설명 |
|------|------|
| $0 | 셸스크립트 이름 |
| $1 | 첫 번째 인자 |
| $2 | 두 번째 인자 |
| $@ | 인자 배열 |
| $# | 인자 개수 |

# 기본값 설정하기

변수가 할당안됐을 기본값을 지정할 수 있다. if문을 줄이려할 때 유용.

```bash
echo Hello, ${NAME-World}! # => Hello, World!
NAME=Alice
echo Hello, ${NAME-World}! # => Hello, Alice!
```

# ref

<https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html>
<http://www.tldp.org/LDP/abs/html/parameter-substitution.html>
<https://github.com/jlevy/the-art-of-command-line/blob/master/README-ko.md>
