# 演習01

## アドホックコマンドを使ってみよう

ansibleにはPlaybookを書かずにモジュールを実行する方法があります。  
それがアドホックコマンドです。  
アドホックコマンドを使えばPlaybookを書かずに素早く実行することが可能になります。  
この演習では、Playbookを書く前に `ansible` コマンドを使ったアドホックコマンドについて動作を確認してみましょう。

アドホックコマンドの詳細については以下を参照ください。

{% embed url="https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html" %}

## Step 1. VM Guestの情報を取得する

まずは、VMのゲスト情報を取得してみましょう。  
VMのゲスト情報を取得するモジュールは [vmware_guest_facts](https://docs.ansible.com/ansible/latest/modules/vmware_guest_facts_module.html#vmware-guest-facts-module) を使用します。  
以下のコマンドを実行してゲスト情報を取得します。  
`-m` で使用するモジュールを指定して `-a` でモジュールに渡すオプションを指定します。

```
ansible localhost -m vmware_guest_facts -a "hostname=127.0.0.1 username=user password=pass validate_certs=no datacenter=DC0 name=DC0_H0_VM0"
```

| モジュールオプション |                   説明                  |
|----------------------|-----------------------------------------|
| hostname             | vCenterまたはESXiのIP/ホスト名          |
| username             | vCenterまたはESXiのアカウント           |
| password             | vCenterまたはESXiのアカウントパスワード |
| validate_certs       | SSL検証の有効/無効                      |
| datacenter           | データセンター名                        |
| name                 | VM名                                    |

問題なく実行できれば、以下のようにVMの情報が取得できます。

```
localhost | SUCCESS => {
    "changed": false,
    "instance": {
        "annotation": null,
        "current_snapshot": null,
        "customvalues": {},
        "guest_consolidation_needed": false,
        "guest_question": null,
        "guest_tools_status": null,
        "guest_tools_version": "0",
        "hw_cluster": null,
        "hw_cores_per_socket": 1,
        "hw_datastores": [
            "LocalDS_0"
        ],
        "hw_esxi_host": "DC0_H0",
        "hw_eth0": {
            "addresstype": "generated",
            "ipaddresses": null,
            "label": "ethernet-0",
            "macaddress": "00:0c:29:6c:9f:78",
            "macaddress_dash": "00-0c-29-6c-9f-78",
            "portgroup_key": "dvportgroup-19",
            "portgroup_portkey": null,
            "summary": "DVSwitch: adf1559c-e6e6-4c06-9f3e-633c9cc6ae82"
        },
        "hw_files": [
            "[LocalDS_0] DC0_H0_VM0/DC0_H0_VM0.vmx"
        ],
        "hw_folder": "/F0/DC0/vm/F0",
        "hw_guest_full_name": null,
        "hw_guest_ha_state": null,
        "hw_guest_id": "otherGuest",
        "hw_interfaces": [
            "eth0"
        ],
        "hw_is_template": false,
        "hw_memtotal_mb": 32,
        "hw_name": "DC0_H0_VM0",
        "hw_power_status": "poweredOn",
        "hw_processor_count": 1,
        "hw_product_uuid": "5606c795-fce7-4238-a4ed-1173cdcf293f",
        "hw_version": "vmx-11",
        "instance_uuid": "fa18c83c-11b5-4dba-a990-8a56291d0af9",
        "ipv4": null,
        "ipv6": null,
        "module_hw": true,
        "snapshots": [],
        "vnc": {}
    }
}
```

次に `name` を `uuid` に変えて実行してみましょう。  
`uuid` には上記で取得した `hw_product_uuid` を指定します。

```
ansible localhost -m vmware_guest_facts -a "hostname=127.0.0.1 username=user password=pass validate_certs=no datacenter=DC0 uuid=5606c795-fce7-4238-a4ed-1173cdcf293f"
```

同じ値が取得できたと思います。  
このように `vmware_guest_facts` では、VM名またはuuidを指定して情報を取得できます

## Step 2. VMの電源を操作する

ここでは、VMの電源操作をしてみましょう。  
VMの電源操作をするモジュールは [vmware_guest_powerstate](https://docs.ansible.com/ansible/latest/modules/vmware_guest_powerstate_module.html#vmware-guest-powerstate-module) を使用します。  
以下のコマンドはVMの電源をOFFにする例です。

```
ansible localhost -m vmware_guest_powerstate -a "hostname=127.0.0.1 username=user password=pass validate_certs=no folder=/F0/DC0/vm/F0 name=DC0_H0_VM0 state=powered-off"
```

| モジュールオプション |                説明                |
|----------------------|------------------------------------|
| folder               | VMが保存されているVMのフォルダパス |
| state                | 電源状態を指定                     |

問題なく実行できれば `CHANGED` と表示されます。  
`hw_power_status` は `poweredOff` になっていることを確認します。

```
localhost | CHANGED => {
    "changed": true,
    "instance": {
(snip)
        "hw_power_status": "poweredOff",
(snip)
```

もう一度同じコマンドを実行してみましょう。

```
ansible localhost -m vmware_guest_powerstate -a "hostname=127.0.0.1 username=user password=pass validate_certs=no folder=/F0/DC0/vm/F0 name=DC0_H0_VM0 state=powered-off"
```

すると `SUCCESS` と表示されると思います。

```
localhost | SUCCESS => {
    "changed": false,
    "instance": {
(sni)
```

これは、Ansibleの `冪等性` によって既に電源OFFになっているVMに対しては処理が実行されないためです。  
このようにAnsibleでは、何回やっても必ず宣言した状態になる `冪等性` が保たれます。  
ただし、全てのモジュールで冪等性が保たれているわけではありません。  
そのため、冪等性が保たれているかは各モジュールのドキュメントまたは実際に動かして確認してください。

それでは、電源をONにしてみましょう。  
電源がOFFになっている状態のVMに対して電源をONにするので `CHANGED` になります。

```
ansible localhost -m vmware_guest_powerstate -a "hostname=127.0.0.1 username=user password=pass validate_certs=no folder=/F0/DC0/vm/F0 name=DC0_H0_VM0 state=powered-on"
```

## 完了

お疲れ様でした。  
これで演習01は終了です。
