---
title: "Git for Windows(Git Bash)にコピペした際「01~」が末尾に付いてくる場合の対処法"
topics: ["Bash", "Windows", "gitbash"]
published: true
---

# 困った

vscodeでGit Bashをデフォルトターミナルにして使っていると、
文字をターミナルにコピペした際末尾に **「01~」** が付いてきてしまう問題が発生。

# 対処法

調査してみるとどうやら`~/.bashrc`に以下を追記すれば良いらしい

```bash
# setting for git bash bug on vscode
bind 'set enable-bracketed-paste off'
```

無事解決しました！

# 他の症状

今回の「01~」が付いてきてしまう問題の他に、
文字数によって **「~」** だったり **「[201~」** が付いてきてしまうケースもあるらしい。

以上。この記事がどなたかのお役に立てれば幸いです。
