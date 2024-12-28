PostgreSQL
===

## インストール

- Redhat系

    - Fedora 41

        - PostgreSQL リポジトリの追加

            ```bash
            sudo dnf install -y \
                https://download.postgresql.org/pub/repos/yum/reporpms/F-$(rpm -E %fedora)-x86_64/pgdg-fedora-repo-latest.noarch.rpm
            ```

        - システムデフォルトの PostgreSQL 無効化

            ```bash
            sudo dnf -qy module disable postgresql
            ```

        - PostgreSQL のインストール

            ```bash
            sudo dnf install -y postgresql17-server
            ```

        - PostgreSQL DB の初期化

            ```bash
            sudo /usr/pgsql-17/bin/postgresql-17-setup initdb
            ```

        - PostgreSQL の起動と自動起動設定

            ```bash
            sudo systemctl enable postgresql-17
            sudo systemctl start postgresql-17
            ```

    - Alma Linux9 / Rocky Linux9

        - インストール可能な PostgreSQL のバージョン確認

            ```bash
            sudo dnf module list postgresql
            ```

        - 使用するバージョンの有効化

            ```bash
            sudo dnf module enable -y postgresql:16
            ```

        - PostgreSQL のインストール

            ```bash
            sudo dnf install -y postgresql-server
            ```

        - PostgreSQL DB の初期化

            ```bash
            sudo postgresql-setup --initdb
            ```

        - PostgreSQL の起動と自動起動設定

            ```bash
            sudo systemctl enable postgresql
            sudo systemctl start postgresql
            ```


- Debian系
    - Debian Linux 12
    - Ubuntu Linux 22.04

        ```bash
        sudo apt install -y postgresql postgresql-contrib
        ```

- サーバ起動設定

    ```bash
    sudo systemctl start postgresql
    sudo systemctl enable postgresql
    ```

## PostgreSQL に接続

- 管理ユーザ (postgres)

    ```bash
    sudo -i -u postgres psql
    ```


## psql コマンド操作

- ユーザ関連

    - ユーザ追加

        - パスワード無し

            ```sql
            CREATE USER ユーザ名;
            ```

        - パスワード付き

            ```sql
            CREATE USER test WITH PASSWORD 'passwd';
            ```

        - パスワード変更・削除

            ```sql
            # 変更
            ALTER ROLE test WITH PASSWORD 'p@ssw0rd';
            # 削除
            ALTER ROLE test WITH PASSWORD null;
            ```

        - testユーザーにデータベース作成権限付与

            ```sql
            ALTER ROLE test WITH createdb;
            ```

    - ユーザ削除

        ```sql
        DROP USER ユーザ名;
        ```

    - ユーザ一覧確認

        ```sql
        \du
        ```

- PostgreSQL プロンプトの終了

    ```sql
    \q
    または
    exit
    ```


## プログラムからの利用

### 準備

- PostgreSQL ユーザ、DB 作成

    ```bash
    sudo -i -u postgres psql
    ```

    ```sql
    CREATE USER test WITH PASSWORD 'passwd';
    CREATE DATABASE testdb;
    GRANT all ON DATABASE testdb TO test;
    \q
    ```

- テーブル作成

    ```bash
    psql -h localhost -U test testdb
    ```

    ```sql
    CREATE TABLE test (
        id SERIAL PRIMARY KEY,
        col1 VARCHAR(50),
        col2 TEXT
    );
    ```

- 作成したテーブルの確認

    |               |                    |
    | :------------ | ------------------ |
    | \d            | テーブル一覧の表示 |
    | \d テーブル名 | テーブル構造の表示 |

- ダミーデータ登録

    ```sql
    INSERT INTO test (col1, col2) VALUES ('テスト1', 'テスト2');
    ```

    ```sql
    SELECT * FROM test;
    ```


### Python

- ライブラリのインストール

    ```bash
    pip3 install SQLAlchemy psycopg2
    ```

- サンプルコード

    ```python
    from sqlalchemy import create_engine, Column, Integer, String, text
    from sqlalchemy.orm import declarative_base, sessionmaker

    username = "test"
    password = "passwd"

    # ベースクラスの作成
    Base = declarative_base()

    # テーブルを表すクラスの定義
    class Test(Base):
        __tablename__ = 'test'
        id = Column(Integer, primary_key=True)
        col1 = Column(String)
        col2 = Column(String)

    if __name__ == "__main__":

        #postgreSQLに接続
        engine=create_engine(f"postgresql://{username}:{password}@localhost:5432/testdb")

        # セッションの作成
        Session = sessionmaker(bind=engine)
        session = Session()

        # データの挿入
        new_record = Test(col1='インサートレコード', col2='インサートデータ')
        session.add(new_record)  # セッションに追加
        session.commit()  # トランザクションを確定

        # 全てのレコードを取得
        results = session.query(Test).all()

        # 結果の出力
        for row in results:
            print(f"id: {row.id}, col1: {row.col1}, col2: {row.col2}")
    ```


## 参考サイト

- [PostgreSQL を Ubuntu に普通にインストール](https://qiita.com/nanbuwks/items/846cf3536a82a2798555)
- [Ubuntu 20.04にPostgreSQLをインストールする方法 [クイックスタート]](https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart-ja)