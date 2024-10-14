Linux モニタリングツール
===

## atop と sysstat

- ツールのインストール

    ```bash
    sudo apt-get update
    sudo apt -y install atop sysstat
    ```

- ログ収集間隔の変更およびディスクと i ノードの使用状況をレポート（-S XALL）を追加

    ```bash
    sudo sed -i 's/^LOGINTERVAL=600.*/LOGINTERVAL=60/' /usr/share/atop/atop.daily
    sudo sed -i -e 's|5-55/10|*/1|' -e 's|every 10 minutes|every 1 minute|' -e 's|debian-sa1|debian-sa1 -S XALL|g' /etc/cron.d/sysstat
    sudo bash -c "echo 'SA1_OPTIONS=\"-S XALL\"' >> /etc/default/sysstat"
    ```

- データ収集開始

    ```bash
    sudo sed -i 's|ENABLED="false"|ENABLED="true"|' /etc/default/sysstat
    sudo systemctl enable atop.service cron.service sysstat.service
    sudo systemctl restart atop.service cron.service sysstat.service
    ```

- スクリプトの修正（cron で周期実行する場合必要らしい）

    ```diff
    --- /usr/lib/sysstat/debian-sa1.origin  2023-01-16 21:02:04.764981715 +0900
    +++ /usr/lib/sysstat/debian-sa1 2023-01-16 21:01:02.040984208 +0900
    @@ -6,7 +6,7 @@
    set -e
    
    # Skip in favour of systemd timer
    -[ ! -d /run/systemd/system ] || exit 0
    +#[ ! -d /run/systemd/system ] || exit 0
    
    # Our configuration file
    DEFAULT=/etc/default/sysstat
    ```

- 参考サイト
    - [EC2 Linux インスタンスのモニタリングツールを設定する](https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-linux-configure-monitoring-tools/)
