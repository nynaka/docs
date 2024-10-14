Windows 小技
===

## Java 関連

- 環境変数

    | 変数      |            値             |
    | :-------- | :-----------------------: |
    | JAVA_HOME | C:\tool\oracle_jdk-21.0.4 |
    | Path      |      %JAVA_HOME%\bin      |

- 環境変数 JAVA_HOME のコマンド設定方法

    - bin フォルダの親フォルダまでのフルパスを設定する。
    - 管理者モードでコマンドプロンプトを追加し、下記のようにコマンドを実行する。

        ```cmd
        setx /m JAVA_HOME "C:\tool\oracle_jdk-21.0.4"
        ```
