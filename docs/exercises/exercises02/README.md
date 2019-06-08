# 演習02

## Playbookを書いてみよう

[演習01](../exercises01/README.md)では、アドホックコマンドを使って情報の取得や状態を変化させました。  
ここでは、Playbookを作ってAnsibleを実行する方法について説明します。

## Step 1. Playbookを作成する

ここでは `gather_vm_guests.yml` というファイルを作成してみましょう。

```
vi gather_vm_guests.yml
```

作成したファイルの先頭に以下の例を追加します。

```yaml
---
- name: GATHER INFOMATION FROM VM GUEST
  hosts: all
  gather_facts: no
```

* `---` は、このファイルがYAMLファイルであるということを示しています。
* `hosts: all` は、このPlaybookを `all` グループに対して実行します。グループ名は次のステップで作成するインベントリフィアルに定義します。
* `gather_facts: no` は、Playbookを実行するホストのOS関連情報を取得しないようにしています。

## Step 2. Inventoryを作成する

Inventoryファイルを作成することで、Playbookを実行する複数ホストの指定やグルーピングをすることができます。  
ここでは `inventory` というファイルを作成してみましょう。

```
vi inventory
```

作成したファイルの先頭に以下の例を追加します。

```
[all]
localhost ansible_connection=local
```

* `[all]` は、Playbookを実行するホストをまとめるグループ名です。
* `localhost` は、Playbookを実行するホストです。何もパラメーターを追加しない場合は、名前解決してアクセスします。
* `ansible_connection` は、ホストへの接続タイプを指定します。ここでは `local` を指定してPlaybookを実行したホスト自身で指定されたPlaybookを実行するようにしています。

{% hint style="info" %} VMwareのモジュールはpyvmomiというPythonのモジュールを使ってAPI経由で実行されます。そのため、直接vCenterへアクセスするのではなく、pyvmomiがインストールされたホストがAPI経由でアクセスします。そのため、接続先はlocalになっています。 {% endhint %}

Ansibleのインベントリに指定できるリモートホストパラメーターの詳細は以下を参照ください。

{% embed url="https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters" %}

## Step 3. Playbookを実行する

それでは、演習01でやったVM Guestの情報をPlaybookを使って取得してみます。

```
vi gather_vm_guests.yml
```

以下のように `tasks` 以下をPlaybookへ追記します。

```yaml
---
- name: GATHER INFOMATION FROM VM GUEST
  hosts: all
  gather_facts: no
  tasks:
    - name: GATHER VM GUEST FACTS
      vmware_guest_facts:
        hostname: 127.0.0.1
        username: user
        password: pass
        validate_certs: no
        datacenter: DC0
        name: DC0_H0_VM0
```

作成したPlaybookを実行してみましょう。

```
ansible-playbook gather_vm_guests.yml -i inventory
```

問題なく実行できれば、以下のように表示されます。

```
PLAY [GATHER INFOMATION FROM VM GUEST] *********************************************************************************************************************

TASK [GATHER VM GUEST FACTS] *******************************************************************************************************************************
ok: [localhost]

PLAY RECAP *************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

アドホックのように取得した値が表示されない？と思った方もいるかもしれません。  
Playbookで取得した値を表示するには、Playbookに表示する処理を書く必要があります。  
結果を表示するには以下のように `register` と `debug` モジュールを追加します。

```yaml
---
- name: GATHER INFOMATION FROM VM GUEST
  hosts: all
  gather_facts: no
  tasks:
    - name: GATHER VM GUEST FACTS
      vmware_guest_facts:
        hostname: 127.0.0.1
        username: user
        password: pass
        validate_certs: no
        datacenter: DC0
        name: DC0_H0_VM0
      register: gather_vm_guest_facts_result

    - debug: var=gather_vm_guest_facts_result
```

* `register` は、モジュールの結果を指定した変数に格納します。ここでは `gather_vm_guest_facts_result` 変数に結果を格納しています。
* `debug` モジュールを使って、格納した変数の中身を表示します。

もう一度Playbookを実行します。

```
ansible-playbook gather_vm_guests.yml -i inventory
```

問題なく実行できれば、結果は以下のように表示されます。

```
PLAY [GATHER INFOMATION FROM VM GUEST] *********************************************************************************************************************

TASK [GATHER VM GUEST FACTS] *******************************************************************************************************************************
ok: [localhost]

TASK [debug] ***********************************************************************************************************************************************
ok: [localhost] => {
    "gather_vm_guest_facts_result": {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "failed": false,
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
}

PLAY RECAP *************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

取得した情報を指定して表示することも可能です。  
例えば、VMの名前だけを表示してみます。  
以下のようにdebug部分を修正します。

```yaml
    - debug: var=gather_vm_guest_facts_result.instance.hw_name
```

もう一度Playbookを実行します。

```
ansible-playbook gather_vm_guests.yml -i inventory
```

問題なく実行できれば、結果は以下のように表示されます。

```
PLAY [GATHER INFOMATION FROM VM GUEST] *********************************************************************************************************************

TASK [GATHER VM GUEST FACTS] *******************************************************************************************************************************
ok: [localhost]

TASK [debug] ***********************************************************************************************************************************************
ok: [localhost] => {
    "gather_vm_guest_facts_result.instance.hw_name": "DC0_H0_VM0"
}

PLAY RECAP *************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## 完了

お疲れ様でした。  
演習02は終了です。
