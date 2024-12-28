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

## プログラミング言語での利用

### C 言語
### Java
### Python

Python で HSM を扱うライブラリには `python-pkcs11` や `PyKCS11` などがあります。

#### PyKCS11 の例

- ライブラリのインストール

    ```bash
    pip install PyKCS11 cryptography
    ```

- 対称鍵を使用した暗号化・複合

    ```python
    import binascii
    import PyKCS11
    from cryptography.hazmat.primitives import padding

    # PKCS#11ライブラリへのパス
    PKCS11_LIB = "/usr/lib/softhsm/libsofthsm2.so"

    # User PIN
    USER_PIN = "4321"

    # 暗号化鍵ラベル
    KEY_LABEL = "TestSymmetricKey"

    # 暗号化対称データ
    plaintext = "Sample Text"

    if __name__ == "__main__":

        # 初期設定
        pkcs11 = PyKCS11.PyKCS11Lib()
        pkcs11.load(PKCS11_LIB)

        # セッションの作成
        slot = pkcs11.getSlotList(tokenPresent=True)[0]  # 最初のスロットを使用
        session = pkcs11.openSession(slot, PyKCS11.CKF_RW_SESSION | PyKCS11.CKF_SERIAL_SESSION)

        # トークンにログイン
        session.login(USER_PIN)

        try:
            # 対称鍵の作成
            key_template = [
                (PyKCS11.CKA_CLASS, PyKCS11.CKO_SECRET_KEY),
                (PyKCS11.CKA_KEY_TYPE, PyKCS11.CKK_AES),
                (PyKCS11.CKA_VALUE_LEN, 32),  # AES-256用の32バイト長
                (PyKCS11.CKA_ENCRYPT, True),
                (PyKCS11.CKA_DECRYPT, True),
                (PyKCS11.CKA_LABEL, KEY_LABEL),
                (PyKCS11.CKA_TOKEN, True),
                (PyKCS11.CKA_PRIVATE, True),
            ]
            key_handle = session.generateKey(key_template)
            print(f"対称鍵を作成しました: {key_handle}")

            # パディングを適用
            print(f"暗号化対称データ: {plaintext}")
            padder = padding.PKCS7(128).padder()  # AESのブロックサイズは128ビット（16バイト）
            padded_plaintext = padder.update(plaintext.encode()) + padder.finalize()

            # 暗号化
            mechanism = PyKCS11.Mechanism(PyKCS11.CKM_AES_ECB, None)
            encrypted_data = session.encrypt(key_handle, padded_plaintext, mechanism)
            print(f"暗号化データ: {binascii.hexlify(bytearray(encrypted_data))}")

            # 復号
            decrypted_data = session.decrypt(key_handle, encrypted_data, mechanism)

            # ckbytelistをbytesに変換
            decrypted_bytes = bytes(decrypted_data)

            # パディングを削除
            unpadder = padding.PKCS7(128).unpadder()
            original_plaintext = unpadder.update(decrypted_bytes) + unpadder.finalize()
            print(f"復号データ: {original_plaintext.decode()}")

            # 鍵の削除
            session.destroyObject(key_handle)
            print("鍵を削除しました。")

        finally:
            # セッション終了
            session.logout()
            session.closeSession()
    ```

### Go
### Rust
