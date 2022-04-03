---
title: "[Vim] Vim 키 매핑으로 단축키 만들어보기"
date: 2022-04-04 01:35 +0900
categories: [Tool, Vim]
tags: [Vim]
---

## 시작하기에 앞서

<img title="" src="/uploads/75196140796f919c69fda04af5c6dfb1abc7008f.png" alt="03-23-38-22-Vim_(text_editor)-Logo.wine.png" width="310" data-align="center">

`Vim`은 그 자체 만으로도 강력한 텍스트 에디터지만 자신만의 단축키를 만들어서 보다 편하게 사용할 수 있습니다.

예를 들어볼까요? `Normal`모드에서 `Insert`모드로 바꾸지 않고 새로운 라인을 변경하려면 어떻게 해야할까요? `o` 혹은 `O`를 눌러서 새로운 라인을 만들고 `Insert`모드로 들어간 뒤에 `ESC`를 눌러서 나오는 방법 말고는 없을 겁니다.

이렇게 자신이 자주 사용하는 기능들을 새로운 단축키로 만들면 얼마나 좋을까요? 오늘은 이러한 단축키를 만드는 방법을 알아보도록 하겠습니다.

## 단축키를 만드는 방법

### `.vimrc` 혹은 `init.vim` 접근하기

**Vim** 단축키를 추가하는 방법 역시, `.vimrc`나 `init.vim`을 수정 및 저장 해주면 됩니다. 이는 사용하는 `Vim` 종류에 따라서 바뀌니 자신에게 맞는 명령어를 입력해주면 됩니다.

- 사용하는 **Vim**이 일반적인 Vim인 경우

  ```bash
  $ vim ~/.vimrc
  ```

- 사용하는 **Vim**이 **Neovim**인 경우

  ```bash
  $ nvim ~/.config/nvim/init.vim
  ```

### 단축키 조합

#### 기본 모습

```vim
map {lhs} {rhs}

" 예제: F3 키를 o를 누른 뒤 ESC를 누르는 것으로 대치
map <F3> o<ESC>
```

**Vim** 키 매핑은 위와 같은 기본 모습을 가집니다. map 이후에는 새로 만들 단축키를 넣어주고 그 이후에는 그 단축키로 매핑될 값을 입력합니다.

#### map의 종류

map은 모드 종류와 `Recursive` 여부에 따라서 수정이 가능합니다.

일단 모드 종류에 따른 매핑 조합을 알아보겠습니다.

| 값      | 모드                                     | 한국어          |
| ------- | ---------------------------------------- | --------------- |
| 빈 공간 | normal, visual, select, operator-pending | 모든 모드       |
| n       | normal mode                              | 노멀 모드       |
| v       | visual, select mode                      | 비주얼 모드     |
| x       | visual mode only                         | 비주얼 모드     |
| s       | select mode only                         | 선택 모드       |
| i       | insert mode                              | 편집 모드       |
| c       | command-line mode                        | 커맨드라인 모드 |
| l       | insert, cmd, RegEx mode                  | Lang-Arg 모드   |
| o       | operator-pending mode                    | Pending 모드    |

여기서 저희가 주로 사용하는 모드는 normal, visual, insert 이므로 **n, v, i**만 생각하고 나머지는 이런게 있구나 정도로만 아시면 될 것 같습니다.

다음은 **Recursive** 여부에 따른 매핑 조합입니다. **Recursive** 여부는 모드 종류 이후에 적습니다.

| 값   | 모드          | 한국어                |
| ---- | ------------- | --------------------- |
| re   | Recursive     | Recursive 매핑        |
| nore | Non-Recursive | Recursive 매핑 안 함. |

Recursive는 재귀하는 것을 의미할텐데 그러면 **Vim**에서의 Recursive는 도대체 뭘까요? 아래 예제로 한 번 확인을 해보겠습니다.

```vim
" Recursive
map o i
map i e
```

위와 같이 매핑을 하면 어떻게 될까요? `o`키는 `i`키를 매핑할거고, `i`키는 `e`키를 매핑하기 때문에 `o -> i -> e`로 최종적으로 `o`키를 눌렀을 때는 `e`키를 눌렀을 때와 동일한 동작을 하게 됩니다.

하지만, 이러한 상황이 아니라 `o`를 눌렀을 때는 `i`를 눌렀을 때 처럼 `Insert` 모드로 들어가고 `i`를 눌렀을 때는 다음 단어로 넘어가는 `e` 기능을 넣고 싶을 떄는 어떻게 해야할까요? 이 때는 바로 **Non-Recursive**하게 키를 매핑해주면 됩니다.

```vim
" Non-Recursive
noremap o i
noremap i e
```

위와 같이 해주면 저희가 원하는 대로 동작할 겁니다!

#### 단축키 조합

그러면 이제 모드 종류와 recursive 종류를 조합한 예를 적겠습니다.

```vim
" Normal 모드에서 Non-Recursive하게 a를 i로 매핑
nnoremap a i

" Insert 모드에서 Non-Recursive하게 Ctrl + n을 :tabnew로 매핑
inoremap <C-n> :tabnew<CR>

" 모든 모드에서 Ctrl-a로 :NERDTreeToggle을 매핑
noremap <C-a> :NERDTreeToggle<CR>
```

이처럼 자신의 입맛에 맞춰서 키를 매핑할 수 있습니다.

#### 특수 키

일반적인 알파벳이나 숫자가 아니라 **Ctrl**이나 **Alt**키와 같이 특수 키들을 매핑하고 싶을 때가 있습니다. 이 때는 아래 표에서 원하는 키를 찾아서 이를 삽입해주면 됩니다.

| 키 명                   | 키 값                                                             |
| ----------------------- | ----------------------------------------------------------------- |
| ESC                     | `<ESC>`                                                           |
| Space                   | `<Space>`                                                         |
| Tab                     | `<Tab>`                                                           |
| Enter                   | `<CR>`                                                            |
| F1~12                   | `<F1> <F2> ... <F12>`                                             |
| Ctrl 조합               | `<C-a> <C-b> ... <C-Z> <C-Space> <C-Tab> ...`                     |
| Alt 조합                | `<A-a> <A-b> ... <A-Z> <A-Space> <A-Tab> ...`                     |
| Shift 조합              | `<S-a> <S-b> ... <S-Z> <S-Space> <S-Tab> ...`                     |
| Ctrl + Alt 조합         | `<C-A-a> <C-A-b> ... <C-A-Z> <C-A-Space> <C-A-Tab> ...`           |
| Ctrl + Alt + Shift 조합 | `<C-A-S-a> <C-A-S-b> ... <C-A-S-Z> <C-A-S-Space> <C-A-S-Tab> ...` |

#### 기타 특수 케이스

1. 조용히 처리를 하고 싶을 때

```vim
nnoremap <C-a> :NERDTreeToggle<CR>
```

위와 같이 **Ctrl + a**를 눌렀을 때 너드 트리가 출력되게 하고 싶다고 합시다. 그리고 직접 이 단축키를 눌러보면 하단 커맨드를 입력하는 부분에 `:NERDTreeToggle`이라고 제가 실행한 단축키가 출력이 됩니다.

<img title="" src="/uploads/90093882df8a824931d3d4710f462bcc059fb6de.png" alt="loading-ag-1128" data-align="center" width="174">

하지만 이런게 마음에 들지 않고 숨기고 싶다면 명령어 조합 전이 `<Silent>`를 입력해주면 됩니다.

```vim
nnoremap <Silent> <C-a> :NERDTreeToggle<CR>
```

2. 키 매핑을 삭제하고 싶을 때

키 매핑을 새로 만드는 것이 아니라 매핑을 취소하고 싶은 경우가 있을 겁니다. 예를 들어, `<C-r>` 키를 다른 용도로 사용하고 있어 기존 용도를 없애버리고 싶거나, 매핑을 없애버리고 싶은 경우, `unmap`을 사용하면 됩니다.

```vim
unmap <C-r>
```

3. 사용자 전용 키

**Vim**에서는 사용자가 원하는 키를 등록하는 `<Leader>`키를 제공하고 있습니다. 기본적으로 `\ (백슬래시키)`로 설정이 되어있으며 다른 특수 키와 동일하게 키 조합으로 사용할 수 있습니다.

```vim
nnoremap <Silent> <Leader>[ :tabprevious<CR>
```

이 `<Leader>`키가 다른 특수 키보다 더 특별한 이유는 키를 백슬래시에서 **다른 키**로 변경할 수 있기 때문입니다, 변경하는 방법은 다음과 같습니다.

```vim
let mapleader="다른 키"


" 예제: leader를 \에서 .로 바꿀 경우
let mapleader="."
```

## 결론

오늘은 이와 같이 Vim 키 매핑을 알아봤습니다. 자신에게 맞는 키 매핑을 적용하여 더욱 효율적인 코드 개발을 하셨으면 좋겠습니다! :)

### 참조

[밤앙개 블로그 - 디자인과 개발 : 네이버 블로그 (naver.com)](https://blog.naver.com/nfwscho/220407221737)
