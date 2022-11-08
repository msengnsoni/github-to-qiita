---
title: "Git for Windows(Git Bash)にstarshipを導入する"
topics: ["Bash", "Windows", "gitbash", "starship"]
published: true
---

# はじめに
本記事はGit for Windows(Git Bash)にstarshipを導入した際の作業備忘録です。
なお、Git Bashはすでに導入済みの環境を前提としています。

# starshipのインストール
今回のインストール用にbinディレクトリを作成しインストールします。
```
$ mkdir -p "$HOME/.local/bin"
$ curl -sS https://starship.rs/install.sh | sh -s -- --bin-dir "$HOME/.local/bin"
```

# インストール後環境設定
starshipをインストールすると以下のように言われます。
* **$PATHにbinディレクトリがないよ**
* **`~/.bashrc`の最終行に`eval "$(starship init bash)"`を追記してね**

なので`~/.bashrc`をこんな感じで編集してあげます。
```bash:~/.bashrc
# starship setting
export PATH=$PATH:/c/Users/username/.local/bin
eval "$(starship init bash)"
```

`source ~/.bashrc`もしくはGit Bashを再起動すると、starshipが適用しているのを確認できました。
![starshipimage.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/6be8e84b-c736-13eb-228b-ac4d3f0c3ed3.png)

やったね！