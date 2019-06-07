# 環境準備

環境はAnsibleをインストールするLinux、VMware ESXi、vCenter Serverを用意する必要があります。  
現在サポートしているLinuxはCentOS/Red Hat Enterprise Linuxです。

## オンプレに環境を準備する

自宅または会社の環境にある物理サーバーにESXiをインストールしてvCenterを用意します。  
自宅で使える検証用ESXi/vCenterのライセンスは [EVALExperience](https://www.vmug.com/vmug2019/membership/evalexperience) で購入することが可能です。  
ライセンス費用は `$200/年` 位です。

環境構築後、Linuxに以下のコンテンツをクローンまたはダウンロードしてください。

**クローンする場合**

```
git clone https://github.com/sky-joker/ansible-vmware-workshops.git
```

**ダウンロードする場合**

```
curl -L https://github.com/sky-joker/ansible-vmware-workshops/archive/master.zip -O
unzip master.zip
```

## vCenter and ESXi API based simulatorを使う

自宅に潤沢なスペックのマシンが無かったり、ESXiやvCenterの準備が出来なかったりライセンスが無い場合はシミュレーターを使って演習が可能です。  
ansibleで使われているシミュレーターはansibleプロジェクトで公開されています。

{% embed url="https://github.com/ansible/vcenter-test-container" %}

ちなみに、シミュレーターはGoで出来ていて以下のプロジェクトで開発されています。

{% embed url="https://github.com/vmware/govmomi/tree/master/vcsim" %}

ディストリビューションごとのシミュレーションデプロイ方法は以下を確認してください。

{% hint style="warning" %}シミューレーターは演習の全てをサポートしていないことをに注意してください。{% endhint %}

### ローカル環境にシミュレーターをデプロイ/削除

#### CentOS7

必要なパッケージをインストールします。

```
yum -y install epel-release && yum -y install git ansible
```

#### RHEL7

以下のリポジトリを有効化します。

```
subscription-manager repos --enable=rhel-7-server-ansible-2.8-rpms --enable=rhel-7-server-extras-rpms
```

必要なパッケージをインストールします。

```
yum -y install git ansible
```

#### シミュレーターデプロイ

[ansible-vmware-workshops](https://github.com/sky-joker/ansible-vmware-workshops) をクローンします。

```
git clone https://github.com/sky-joker/ansible-vmware-workshops.git
```
 シミュレーターをデプロイするために `provisioner` ディレクトリへ移動します。

```
cd ansible-vmware-workshops/provisioner/
```

ローカル環境にシミュレーターをデプロイします。

```
ansible-playbook provision_lab.yml -i inventory -l localhost
```

デプロイが完了したら以下のコマンドを実行して動作確認をします。

```
curl -sk https://user:pass@127.0.0.1/about
```

上記のコマンドを実行するとサポートしているメソッド一覧が表示されます。

```
{
  "Methods": [
    "AcquireCloneTicket",
    "AcquireGenericServiceTicket",
    "AddAuthorizationRole",
    "AddCustomFieldDef",
    "AddDVPortgroup_Task",
    "AddHost_Task",
(nip)
  "Types": [
    "AuthorizationManager",
    "ClusterComputeResource",
    "ComputeResource",
    "CustomFieldsManager",
    "Datacenter",
(snip)
```

シミュレーターでは、上記で表示されたメソッドのみサポートされます。

#### シミュレーター削除

ローカル環境のシミュレーターを削除します。

```
ansible-playbook teardown_lab.yml -i inventory -l localhost
```
