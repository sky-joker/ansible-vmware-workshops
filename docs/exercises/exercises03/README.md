# 演習03

## モジュールのドキュメントを確認してみよう

Ansibleの標準モジュールの仕様やオプションについて確認する方法は主に以下の2通りが考えられます

* 公式のドキュメント
* コマンドで確認

## 公式ドキュメントで確認

最新のドキュメントは以下のページから確認できます。

{% embed url="https://docs.ansible.com/ansible/latest/modules/modules_by_category.html" %}

VMwareのモジュールは以下のページから確認できます。

{% embed url="https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#vmware" %}

## コマンドで確認

公式ドキュメントが見れない場合やすぐ確認したい時はコマンドでモジュールのドキュメントを確認することが出来ます。  
コマンドは `ansible-doc` を使用します。  
それでは `vmware_guest_facts` モジュールのドキュメントを確認してみましょう。

```
ansible-doc vmware_guest_facts
```

上記コマンドを実行すると以下のようにドキュメントが確認出来ます。

```
> VMWARE_GUEST_FACTS    (/usr/lib/python2.7/site-packages/ansible/modules/cloud/vmware/vmware_guest_facts.py)

        Gather facts about a single VM on a VMware ESX cluster.

  * This module is maintained by The Ansible Community
OPTIONS (= is mandatory):

= datacenter
        Destination datacenter for the deploy operation


- folder
        Destination folder, absolute or relative path to find an existing guest.
        This is required if name is supplied.
        The folder should include the datacenter. ESX's datacenter is ha-datacenter
        Examples:
           folder: /ha-datacenter/vm
           folder: ha-datacenter/vm
           folder: /datacenter1/vm
           folder: datacenter1/vm
           folder: /datacenter1/vm/folder1
           folder: datacenter1/vm/folder1
           folder: /folder1/datacenter1/vm
           folder: folder1/datacenter1/vm
           folder: /folder1/datacenter1/vm/folder2
           folder: vm/folder2
           folder: folder2
        [Default: (null)]
(snip)
```

`ansible-doc` コマンドに仕様やオプションなどを確認をしたいモジュール名を指定することでCLIでもドキュメントが見れることが分かりました。

## 完了

お疲れ様でした。  
演習03は終了です。
