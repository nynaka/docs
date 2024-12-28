SoftHSM
===

## インストール

- Redhat系 (Fedora、Alma Linux9、Rocky Linux9)

    ```bash
    sudo dnf install -y softhsm opensc
    ```

- Debian系 (Debian Linux 12、Ubuntu Linux 22.04)

    ```bash
    sudo apt install -y softhsm2 opensc
    ```

## pkcs11-tool での SoftHSM の操作

- ライブラリパスの設定

    ライブラリパスを環境変数に設定しておくと少しだけ便利です。

    ```bash
    export LIBPATH=/usr/lib/softhsm/libsofthsm2.so
    ```

- サポートアルゴリズムの確認

    ```bash
    sudo pkcs11-tool --module $LIBPATH -M
    ```

- トークンの初期化

    ```bash
    sudo pkcs11-tool --module $LIBPATH \
        --init-token \
        --label "init-token" \
        --so-pin 1234
    ```

- user pin の設定

    ```bash
    sudo pkcs11-tool --module $LIBPATH \
        --init-pin \
        --so-pin 1234 \
        --pin 4321
    ```

- スロットの確認

    ```bash
    sudo pkcs11-tool --module $LIBPATH -L
    ```

- 鍵の作成

    - 対象鍵の作成

        ```bash
        sudo pkcs11-tool --module $LIBPATH \
            --keygen \
            --key-type AES:32 \
            --label "aeskey" \
            --login --pin 4321
        ```

    - RSA 非対称鍵の作成

        ```bash
        sudo pkcs11-tool --module $LIBPATH \
            --keypairgen \
            --key-type rsa:2048 \
            --label "rsakey" \
            --login --pin 4321
        ```

    - ECDSA 非対称鍵の作成

        ```bash
        sudo pkcs11-tool --module $LIBPATH \
            --keypairgen \
            --key-type EC:prime256v1 \
            --label "ecdsakey" \
            --login --pin 4321
        ```
