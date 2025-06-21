# Django web アプリケーション開発環境の設計

## 仮想環境の設計

各プロジェクト毎に、仮想環境を作成する。

- 1アプリケーションを1プロジェクトとする
- プロジェクトごとに仮想環境を作する
- 仮想環境毎にdjangoや必要なライブラリをインストールする
```
~/web_dev/
├── project_name1/
│   ├── venv/                   ← 仮想環境
│   ├── manage.py
│   ├── config/           ← settings.py, wsgi.py,
│   ├── app_name/
│   └── requirements.txt
├── project_name2/
│   ├── venv/
│   ├── manage.py
│   ├── config/
│   ├── app_name/
│   └── requirements.txt
```


### ① プロジェクトのルートディレクトリ作成

```bash
mkdir -p ~/web_dev/project_name1
cd ~/web_dev/project_name1
```

---

### ② 仮想環境を作成

```bash
python3 -m venv venv
source venv/bin/activate
```

---

### ③ 仮想環境内に Django をインストール

```bash
pip install django
```

---

### ④ プロジェクト作成（`manage.py`などをここに配置）

```bash
django-admin startproject config .
```

> `.` を付けることで、`manage.py` を **現在のディレクトリ（project\_name1）直下に作成する**
> そうしないと `project_name1/projectname1/manage.py` のようにネストが深くなってしまう

---

### ⑤ 必要に応じてアプリ作成、モジュールをインストール

```bash
python manage.py startapp app_name
```

```bash
pip install gunicorm
```


---

### ⑥ 依存パッケージを記録

```bash
pip freeze > requirements.txt
```

## 本番環境と開発環境の差異について
### 本番環境と開発環境の構成
#### 本番環境
- Nginx + Gunicorn + Django + PostgrSQL
- Docker + Nginx + Gunicorn + Django + PostgrSQL

#### 開発環境
- runserver + Djanogo + PostgreSQL
- Docker + Nginx + Gunicorn + Django + PostgrSQL

## 開発環境の設計思想について

### 開発環境にPostgreSQLを採用した理由と背景
* PostgreSQLとMySQLでは、SQLの書き方・型・制約に**非互換**がある。
  例：

  * `AutoField`の初期値の扱い
  * `TextField`や`BooleanField`のバインディング
  * トランザクション挙動の違い
* 本番でPostgreSQLを使うなら、**開発でもPostgreSQLを使うのがセオリー**。

### 開発環境にNginxとGunicornを採用しない理由と背景

#### ① 開発サイクルにおいて非効率だから

* `runserver`はコード変更時に自動リロードが効く。
  → `Gunicorn`だと毎回プロセス再起動が必要。
* NginxやGunicornの設定ファイルをちょくちょく触るのは冗長。
* デバッグが面倒（Gunicorn + Nginxのログを個別に見る必要がある）。

---

#### ② Nginxはローカル開発には**不要なレイヤー**

* 開発段階でNginxの**リバースプロキシ・SSL・キャッシュ制御**などの機能は大抵使わない。
* そのため、開発中のNginxは\*\*「ただの面倒なゲートキーパー」\*\*になりやすい。

---

#### ③ ローカルでNginxをポート80で動かすにはroot権限が必要

* `sudo`を使う必要があり、開発スピードを損なう。
* OSによってNginxのインストール、起動方法が違い、面倒。

---

#### ④ WindowsやmacOSでは環境構築がさらに面倒

* Nginxの設定や挙動がLinuxと異なることも多く、トラブルのもと。
* 特に本番をUbuntuで運用するなら、**ローカルでの完全再現が困難**。
