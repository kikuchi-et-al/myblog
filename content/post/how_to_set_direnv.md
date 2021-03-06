---
title: "direnvの設定方法"
date: 2020-01-02T16:34:37+09:00
draft: false
---

direnvとはディレクトリ毎に環境変数を定義し特定のディレクトリに移動したときのみ環境変数を有効化できるツールである。

例えばAWSアカウントのアクセスキー/シークレットキーの情報をディレクトリ毎を設定することで、ディレクトリに移動してきたタイミングで自動でこれらのキーを環境変数として設定することができ、AWS CLIを実行できる環境を素早く用意することができる。

逆にdirenvを設定しないとAWSアカウントの切り替えの度に環境変数を設定し直す必要がでてくる。

この記事ではディレクトリ毎にAWSアカウントのアクセスキー/シークレットキーの情報を登録し、実際にAWS CLIを実行できる環境を用意する方法を説明する。

<!--more-->

### 1. brewコマンドでdirenvをインストール
```
$ brew install direnv
```
### 2. whichコマンドdirenvの存在確認
```
$ which direnv
/usr/local/bin/direnv
```
### 3. ホームディレクトリにある .bashrc を開く
```
$ vi ~/.bashrc
```
### 4. .bashrcの最下行に下記を追加
```
eval "$(direnv hook bash)"
```
### 5. 設定反映
```
$ source ~/.bashrc
```
### 6. ディレクトリを作成して移動
```
$ mkdir ~/direnv
$ cd direnv
```
### 7. aws configureを設定
```
$ aws configure --profile AWSアカウント名
```
### 8. アクセスキー・シークレットキーを入力
【入力例】
```
AWS Access Key ID [None]:xxxxxxxxxx
AWS Secret Access Key [None]:yyyyyyyyyy
Default region name [None]:ap-northeast-1 ←<そのままenter> 入れてもOK
Default output format [None]:ここは空欄でもOK
```
### 9. 情報を取得できるか確認
下記のように出力されていればOK。
```
$ aws sts get-caller-identity --profile AWSアカウント名
{
    "Account": "アカウントID",
    "UserId": "XXXXXXXXXXXXXXX",
    "Arn": "arn:aws:iam::アカウントID:user/IAMユーザー名"
}
```
### 10. アカウント用のディレクトリを作成し移動
```
$ mkdir AWSアカウント名
$ cd AWSアカウント名
```
### 11. アカウント用のディレクトリに移動後.envrc を作成
```
$ vi .envrc
```
### 12. .envrcの中に下記内容を入力

```
export AWS_DEFAULT_PROFILE=手順7で設定したAWSアカウント名
export AWS_PROFILE=$AWS_DEFAULT_PROFILE
export AWS_REGION=リージョン名
```
### 13. .envrcの設定が完了すると下記メッセージが表示される

```
direnv: error .envrc is blocked. Run `direnv allow` to approve its content.
```
`direnv allow` と入力し下記のように表示されればOK
```
$ direnv allow
direnv: loading .envrc
direnv: export +AWS_DEFAULT_PROFILE +AWS_PROFILE +AWS_REGION
```
### 14. もう一度ホームディレクトリに戻り再度アカウント用ディレクトリに移動
【入力例】
```
$ cd ~/
direnv: unloading
$ cd ~/direnv/AWSアカウント名/
direnv: loading .envrc
direnv: export +AWS_DEFAULT_PROFILE +AWS_PROFILE +AWS_REGION
```
上記のように表示されればOK
### 15. 動作確認
`aws sts get-caller-identity` コマンドで設定内容が意図したものであるか確認する。

CLI実行前には `aws ec2 describe-instances --region ap-northeast-1` コマンド等で対象アカウント内のリソース情報が表示されていることを確認し、作業対象が間違っていないことを必ず確認すること。
