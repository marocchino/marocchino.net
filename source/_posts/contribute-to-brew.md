---
title: brew에 기여하기
date: 2021-02-23 07:49:02
tags:
- macos
- homebrew
---

## 동기

굉장히 드물게 homebrew에 최신버전이 없는걸 내가 젤 먼저 눈치챌 때가 있다.
새버전 나올때 까지 기다리는 것도 방법이긴 한데 의외로 기여하는 방법이 어렵지
않으니 어떻게 해야하는지 알고 있다면 직접 할 수도 있다.

## 발견

내 경우는 배포처에서 새버전을 만들면서 기존 버전을 지워서 눈치챌 수 있었다.

```bash
❯ brew install texshop
==> Downloading https://pages.uoregon.edu/koch/texshop/texshop-64/texshop458.zip
-#O#- #   #
curl: (22) The requested URL returned error: 404 Not Found
Error: Download failed on Cask 'texshop' with message: Download failed: https://pages.uoregon.edu/koch/texshop/texshop-64/texshop458.zip
```

## 한일

먼저 edit로 해당 탭을 열어 버전만 수정해서 인스톨 해본다.

```bash
❯ brew edit texshop
Editing /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask/Casks/texshop.rb
❯ brew install texshop
==> Downloading https://pages.uoregon.edu/koch/texshop/texshop-64/texshop459.zip
######################################################################## 100.0%
Error: SHA256 mismatch
Expected: 97ccf1ebfbb30b8557c972ee2a207a4b570057ae127b9b93cfada73ffbdc907e
  Actual: 5c018481244098c0d622f9b976cd0af1f923c7a9e80ab6fe3969ad4b338334fa
    File: /Users/marocchino/Library/Caches/Homebrew/downloads/03b3398e2bc593405fc7eeaa245c15cf0e9cfbf0a8882b5d0e6d7e2ac9b08c5e--texshop459.zip
To retry an incomplete download, remove the file above.
```

sha가 다르다고 통과시켜주지 않으니 저것도 고쳐주자.

```bash
~ took 1m41s ❯ brew edit texshop
Editing /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask/Casks/texshop.rb
~ took 10s ❯ brew install texshop
Updating Homebrew...
==> Auto-updated Homebrew!
Updated 2 taps (homebrew/core and homebrew/cask).
==> Updated Formulae
Updated 1 formula.
==> Updated Casks
Updated 2 casks.

==> Downloading https://pages.uoregon.edu/koch/texshop/texshop-64/texshop459.zip
Already downloaded: /Users/marocchino/Library/Caches/Homebrew/downloads/03b3398e2bc593405fc7eeaa245c15cf0e9cfbf0a8882b5d0e6d7e2ac9b08c5e--texshop459.zip
==> Installing Cask texshop
==> Moving App 'TeXShop.app' to '/Applications/TeXShop.app'
🍺  texshop was successfully installed!
```

설치 해서 문제없이 프로그램이 열리는 걸 확인했으면 이제 방금 수정한 내용으로
[PR](https://github.com/Homebrew/homebrew-cask/pull/100249)을 던진다.

탬플릿을 볼 수 있는데 밑에 새캐스크 만드는 법을 무시하면 내가해야할 일을 3가지
이다.

> **Important:** *Do not tick a checkbox if you haven’t performed its action.*
> Honesty is indispensable for a smooth review process.
>
> After making all changes to a cask, verify:
>
> - [ ] The submission is for
    [a stable version](https://github.com/Homebrew/homebrew-cask/blob/master/doc/development/adding_a_cask.md#stable-versions)
    or
    [documented exception](https://github.com/Homebrew/homebrew-cask/blob/master/doc/development/adding_a_cask.md#but-there-is-no-stable-version).
> - [ ] `brew audit --cask {{cask_file}}` is error-free.
> - [ ] `brew style --fix {{cask_file}}` reports no offenses.

시키는 대로 하자.

```bash
❯ brew audit --cask texshop
==> Installing 'bundler' gem
... 생략 ...
Bundle complete! 30 Gemfile dependencies, 82 gems now installed.
Bundled gems are installed into `../../usr/local/Homebrew/Library/Homebrew/vendor/bundle`
Post-install message from sorbet:

  Thanks for installing Sorbet! To use it in your project, first run:

    bundle exec srb init

  which will get your project ready to use with Sorbet.
  After that whenever you want to typecheck your code, run:

    bundle exec srb tc

  For more docs see: https://sorbet.org/docs/adopting
audit for texshop: passed
❯ brew style --fix texshop

1 file inspected, no offenses detected
```

CI가 돌고 문제가 없으면 바로 머지된다. (8분 밖에 걸리지 않았다!)
