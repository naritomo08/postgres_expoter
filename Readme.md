# postgres_expoter

ansibleでpodman/Postgre_Expoterを導入するPlaybook

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
pg_user: "pgexp"
pg_port: 5432

監視対象DBのアクセス許可ファイルパス
postgres_pg_hba_path: "/var/lib/pgsql/data/pg_hba.conf"
```

## 起動方法

### postgres_Exporter導入

```
#　導入後監視対象DB設定変更やExpoterバージョンしたい際は一度ロールバックして
　　ソース更新して作成すること。

# Exporterは一度接続されるとDBユーザー消しても接続状態が維持されるため、
  DBユーザー削除と合わせて行うこと。

# ★ 導入
ansible-playbook -i inventory.ini pgexp_install.yml --ask-vault-pass
# └── deploy_state=present (既定)
Vaultパスワードいれる。

# ★ ロールバック
ansible-playbook -i inventory.ini pgexp_install.yml --ask-vault-pass \
  --extra-vars "deploy_state=absent"
Vaultパスワードいれる。

# ★ Exporterバージョン変更しての再度導入（Postgres_Expoterバージョン変更も可）
ansible-playbook -i inventory.ini pgexp_install.yml --ask-vault-pass \
  --extra-vars 'deploy_state=present exporter_version=v0.16.0'
Vaultパスワードいれる。
```

### postgresSQL設定
```
# DBユーザーを再度変更する際は一旦ロールバックしてソース更新して再設定すること。

# ★ 設定
ansible-playbook -i inventory.ini postgres_setup.yml --ask-vault-pass
# └── deploy_state=present (既定)
Vaultパスワードいれる。

# ★ ロールバック
ansible-playbook -i inventory.ini postgres_setup.yml --ask-vault-pass \
  --extra-vars "deploy_state=absent"
Vaultパスワードいれる。
```

## 動作確認

```
ANSIBLE_STDOUT_CALLBACK=oneline \
ansible-playbook -i inventory.ini pgexp_check.yml

以下の結果が出ること。
almatest3 | SUCCESS => {    "changed": false,    "msg": "✅ pg_up が 1 です"}

```