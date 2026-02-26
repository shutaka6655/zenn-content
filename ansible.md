# APIがないサーバーをどう自動化する？SSH地獄からAnsibleに逃げた話

## 3行まとめ

- APIがなくても、SSHさえあれば自動化はできる  
- Ansibleを使えば、対象側の開発なしで安全に構成変更できる  
- さらにPythonから制御すれば「アプリから外部端末を操作する基盤」になる  

---

## APIがない。どうする？

APIがあるなら話は早い。

HTTPで叩くだけ。  
レスポンスも取れる。  
エラーもハンドリングできる。

最高。

でも、こういうケースありませんか？

- ただのLinuxサーバー
- ラズパイ
- とりあえず置いてある検証機
- 昔からあるオンプレ機

APIなんてない。

じゃあどうする？

---

## とりあえずSSH…の未来

やりがちなのがこれ。

- sshで入る
- scpでファイル送る
- コマンド叩く

一台ならいい。

でも台数が増えた瞬間に始まる。

### SSH地獄

- 3台中1台だけ失敗
- どこで失敗したか分からない
- 再実行すると二重更新
- 差分があるのかも分からない
- ログはgrep頼り

そして思う。

> これ、将来事故るな…

---

## そこでAnsible

ここで登場するのが **Ansible**。

- SSH接続できる
- Pythonが入っている（ほぼ標準）

これだけで動く。

エージェント不要。  
対象側の開発不要。

つまり、

> 既存サーバーをそのまま自動化できる

---

## 何が違うのか？

### ① 冪等性がある

- 変更がなければ何もしない  
- 必要な変更だけ行う  

毎回上書きするシェルとは違う。

何度でも安全に実行できる。

---

### ② 成功・失敗・到達不能が分かれる

- 成功
- タスク失敗
- SSH接続不可（unreachable）

が明確に分かれる。

どのホストの、どのタスクで止まったかも出る。

ログ設計を自分で書かなくていい。

---

### ③ 複数台が前提設計

- 開発環境
- 検証環境
- 本番環境

をグループ化できる。

全台一斉も、特定だけも、1台だけも可能。

台数が増えるほど差が出る。

---

## 実際のコード（最小例）

今回は、

- 外部端末にファイルを作成
- それをPythonから実行

してみる。

CLIではなく、PythonからAnsibleを実行する。ここが今回のポイント。  
つまり、**アプリケーションから安全に外部端末を制御できる**

ということ。

### インストール

```bash
pip install ansible ansible-runner
```

### ディレクトリ構成
.  
├── sample.py   : 実行スクリプト  
└── sample.yaml : play-bookファイル

### pythonコード



``` python
import ansible_runner

r = ansible_runner.run(
    private_data_dir='/aaa/bbb', # sample.pyを置いたディレクトリ
    playbook='sample.yaml',
    inventory={
        "dev-servers": {
            "hosts": {
                "pc-1": {
                    "ansible_host": "192.168.xxx.xxx",
                    "ansible_connection": "ssh",
                    "ansible_user": "user",
                    "ansible_password": "password",
                    "ansible_become_password": "password"
                }
            }
        }
    }
)

print(f"Status: {r.status}")
print(f"RC: {r.rc}")

for event in r.events:
    if event.get("event") == "runner_on_unreachable":
        print("接続失敗:", event["event_data"]["res"])

    elif event.get("event") == "runner_on_failed":
        print("タスク失敗:", event["event_data"]["res"])
```
※ パスワードのベタ書きは本番では避けてください。


### yamlコード
``` yaml
- name: 設定ファイルを作成
  hosts:
    - dev-servers
  become: true
  tasks:
    - name: test.txt を作成
      copy:
        content: "HELLO"
        dest: "/var/www/test.txt"
```
これだけ。  
実行すると /var/www/test.txt が生成される。


## 失敗もちゃんと分かる

Ansibleは「実行できる」だけではない。  
**失敗も構造化して取得できる。**

- タスク失敗 → `runner_on_failed`
- SSH接続不可 → `runner_on_unreachable`

つまり、

- どこで  
- 何が原因で  
- どのホストが  

失敗したのかが、明確に取れる。

SSHスクリプト時代とは安心感が違う。

---

## これが本質

APIがなくてもいい。

SSHがあればいい。

その上にAnsibleを乗せるだけで、  
外部端末は「制御可能な資産」に変わる。

そしてそれをPythonから呼び出せば、

- Webアプリからサーバー設定変更
- バッチから一斉更新
- 管理画面から構成反映

までできる。

---

## まとめ

APIがない外部端末をどう扱うか？

- 無理やりAPIを作るか
- SSHスクリプトで頑張るか
- それとも仕組みで解決するか

自分はSSH地獄から逃げた。

Ansibleは「構成管理ツール」だけじゃない。  
APIがない世界を制御可能にする現実解だと思っている。