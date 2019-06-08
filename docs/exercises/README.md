# 演習

演習を始める前に以下の情報を確認してください。

* vCenterのIPアドレスまたはホスト名
* vCenterのアカウント・パスワード

## シミュレーターを使う場合

シミュレーターを使う場合は、以下の方法で各情報が取得できます。

### 全オブジェクトパス

**コマンド**

```
curl -sk http://127.0.0.1:5000/govc_find
```

**結果例**

```
[
  "/",
  "/F0",
  "/F0/DC0",
  "/F0/DC0/vm",
  "/F0/DC0/vm/F0",
  "/F0/DC0/vm/F0/DC0_H0_VM0",
  "/F0/DC0/vm/F0/DC0_H0_VM1",
  "/F0/DC0/vm/F0/DC0_C0_RP0_VM0",
  "/F0/DC0/vm/F0/DC0_C0_RP0_VM1",
(snip)
]
```

### 全オブジェクトパスのフィルタリング

`govc_find` では以下のように `?filter=` を指定してフィルタリングをかけることができます。  
フィルターに使用できる文字列は以下の `フィルタリング` 列にあるものです。

| フィルタリング |             説明            |
|----------------|-----------------------------|
| VA             | VirtualApp                  |
| CCR            | ClusterComputeResource      |
| DC             | Datacenter                  |
| F              | Folder                      |
| DVP            | DistributedVirtualPortgroup |
| H              | HostSystem(ESXi)            |
| VM             | VirtualMachine              |
| N              | Network                     |
| ON             | OpaqueNetwork               |
| RP             | Resource Pool               |
| CR             | ComputeResource             |
| D              | Datastore                   |
| DVS            | DistributedVirtualSwitch    |

{% embed url="https://github.com/vmware/govmomi/blob/master/govc/USAGE.md#find" %}

**コマンド**

```
curl -sk http://127.0.0.1:5000/govc_find?filter=DC
```

**結果例**

```
[
  "/F0/DC0"
]
```

### VMの情報

**コマンド**

```
curl -sk http://127.0.0.1:5000/govc_vm_info
```

**結果例**

```
{
  "DC0_C0_RP0_VM0": {
    "Boot time": "2019-06-07 14:30:07.240733938 +0000 UTC",
    "CPU": "1 vCPU(s)",
    "Guest name": "otherGuest",
    "Host": "DC0_C0_H2",
    "IP address": "",
    "Memory": "32MB",
    "Name": "DC0_C0_RP0_VM0",
    "Path": "/F0/DC0/vm/F0/DC0_C0_RP0_VM0",
    "Power state": "poweredOn",
    "UUID": "292d2e87-1d83-4186-afcf-6793fcbfa0ee"
  },
(snip)
}
```

### ホストの情報

**コマンド**

```
curl -sk http://127.0.0.1:5000/govc_host_info
```

**結果**

```
{
  "DC0_C0_H0": {
    "Boot time": "2019-06-07 14:30:07.237090714 +0000 UTC",
    "CPU usage": "67 MHz (0.7%)",
    "Logical CPUs": "4 CPUs @ 2294MHz",
    "Manufacturer": "VMware, Inc. (govmomi simulator)",
    "Memory": "4095MB",
    "Memory usage": "1404 MB (34.3%)",
    "Name": "DC0_C0_H0",
    "Path": "/F0/DC0/host/F0/DC0_C0/DC0_C0_H0",
    "Processor type": "Intel(R) Core(TM) i7-3615QM CPU @ 2.30GHz",
    "State": "connected"
  },
(snip)
}
```
