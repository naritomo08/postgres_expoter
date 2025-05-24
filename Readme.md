# postgres_expoter

ansibleでDockerCompose/Postgre_Expoterを導入するPlaybook

ansibleバージョンが2.15以上であること。
```
ansible --version
```

以下のコマンドで指定バージョンにアップグレード可能
```
python -m pip install --upgrade pip
python -m pip install "ansible-core>=2.15,<2.16" --user
```

## 監視用DBアカウント作成

DBサーバにログイン
ここで設定したユーザ、パスは控える。
```
sudo -i
su - postgres
psql
CREATE ROLE pgexp WITH LOGIN PASSWORD 'S3curePW';
GRANT pg_monitor TO pgexp;
Ctrl + Cで抜ける
```

## DBへのホストアクセス許可追加

```
sudo vi /var/lib/pgsql/data/pg_hba.conf

最終行に追記
host    all             all             172.18.0.0/16            md5

sudo systemctl reload postgresql
```

## DBパスワード設定

以下の方法でPostgres接続パスワードの暗号化を行う。

```
ansible-vault encrypt_string 'DBパスワード' --name pgexp_pass > group_vars/secret.yml
2回Vaultパスワードいれる。
ここのパスワードはplaybook作成時に使用する。

all.ymlの最終行に内容を記載して、作成したファイルは削除する。
```

## DB設定確認

以下の設定ファイルを確認し、動かすDB設定に合わせる。
```
vi groups_vars

以下の変数内容を確認する。
必要に応じ変更する。

監視対象のDBホスト、パスワード設定
pg_host: "172.17.0.1"
pg_port: 5432

監視対象DBのアクセス許可ファイルパス
postgres_pg_hba_path: "/var/lib/pgsql/data/pg_hba.conf"
```

## 起動方法

```
# ★ 作成（初回／再導入）
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
# └── deploy_state=present (既定)
Vaultパスワードいれる。

# ★ ロールバック（いつでも取り消し）
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass \
  --extra-vars "deploy_state=absent"
Vaultパスワードいれる。

# ★ バージョン変更しての再度導入（Postgres_Expoterバージョン変更も可）
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass \
  --extra-vars 'deploy_state=present exporter_version=v0.16.0'
Vaultパスワードいれる。
```

## 動作確認

```
curl http://192.168.11.10:9187/metrics | grep pg_up

→　"pg_up 1"が返ってくること。
```