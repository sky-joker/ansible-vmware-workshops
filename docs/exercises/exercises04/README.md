# 演習03

## レポートを作成してみよう

ここでは、Ansibleと[Jinja2](http://jinja.pocoo.org/docs/2.10/)を使ったレポートの動的作成方法について説明します。  
Jinja2でレポートの元となるテンプレートを作成しておき、それを使ってレポートを生成することが出来ます。  
テンプレートは自由に作成できるためMarkdownやcsvといった形式で作成することも可能です。

## Step 1. レポート生成するファイルの確認する

本演習ディレクトリへ移動します。

```
cd ansible-vmware-workshops/docs/exercises/exercises04/
```

移動したディレクトリに `report_generate_vmware.yml` と `report_template_csv.j2` が存在していることを確認します。

```
ls
README.md  inventory  report_generate_vmware.yml  report_template_csv.j2
```

## Step 2. レポート生成するファイルの説明

`report_generate_vmware.yml` の中身を見て見ましょう。

```
cat report_generate_vmware.yml
```

```yaml
---
- name: GENERATE REPORT FOR VMS
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
        name: "{{ item }}"
      loop:
        - DC0_H0_VM0
        - DC0_H0_VM1
      register: gather_vm_guest_facts_result

    - name: GENERATE REPORT
      template:
        src: report_template_csv.j2
        dest: vms_report.csv
```

* VM情報を取得する処理に `loop` が追加されています。loopは、そのタスクを要素分だけ繰り返し実行します。ここではVM2台に対して情報を取得します。loopで回す時は展開するための変数を指定する必要があります。デフォルトは `"{{ item }}"` です。
* `template` モジュールを使って、レポートを生成します。`src` にはテンプレートの元になるファイルを指定し `dest` にはアウトプットするファイル名を指定します。

次に `report_template_csv.j2` の中身を見て見ましょう。

```
cat report_template_csv.j2
```

```
vmname,folder,vmtools_status,vmtools_version,cpu socket num,cpu core num,memory,powere,hw version,uuid,instanceuuid,
{% for vm in gather_vm_guest_facts_result.results %}
{{ vm.instance.hw_name}},{{ vm.instance.hw_folder }},{{ vm.instance.guest_tools_status }},{{ vm.instance.guest_tools_version }},{{ vm.instance.hw_cores_per_socket }},{{ vm.instance.hw_processor_count }},{{ vm.instance.hw_memtotal_mb }},{{ vm.instance.hw_power_status }},{{ vm.instance.hw_version }},{{ vm.instance.hw_product_uuid }},{{ vm.instance.instance_uuid }}
{% endfor %}
```

* 1行目はカラムの名前です。
* 2行目以降は `for` を使ってVM情報が格納されている `gather_vm_guest_facts_result` の中からレポートに必要な変数を抽出しています。

Jinja2を使ったテンプレートは、このようにコードを書いておくことで自由なレポートを生成することが出来ます。

## Step 3. レポートを生成する

以下のコマンドを実行してレポートを生成してみましょう。

```
ansible-playbook report_generate_vmware.yml -i inventory
```

問題なく実行できれば、結果は以下のように表示されます。

```
PLAY [GENERATE REPORT FOR VMS] *****************************************************************************************************************************

TASK [GATHER VM GUEST FACTS] *******************************************************************************************************************************
ok: [localhost] => (item=DC0_H0_VM0)
ok: [localhost] => (item=DC0_H0_VM1)

TASK [GENERATE REPORT] *************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

生成されたレポートを確認してみましょう。

```
cat vms_report.csv
```

csv形式でレポートが生成されていることが確認できます。

```
vmname,folder,vmtools_status,vmtools_version,cpu socket num,cpu core num,memory,powere,hw version,uuid,instanceuuid,
DC0_H0_VM0,/F0/DC0/vm/F0,,0,1,1,32,poweredOn,vmx-11,5606c795-fce7-4238-a4ed-1173cdcf293f,fa18c83c-11b5-4dba-a990-8a56291d0af9
DC0_H0_VM1,/F0/DC0/vm/F0,,0,1,1,32,poweredOn,vmx-11,75d12a24-34b6-4db3-a5e6-bc12af9f3002,b2157668-ade3-4e38-86b1-4275dc5deb11
```

これをExcelで表示して編集することも可能です。  
このようにAnsibleでは動的なレポート作成も自動化できます。  
棚卸しや構成の状態を確認したい時などもAnsibleが役に立ちます。

## 完了

お疲れ様でした。  
演習04は終了です。
