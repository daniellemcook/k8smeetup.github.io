---
approvers:
- erictune
- jbeda
title: VMware vSphere
---


本页面涵盖了如何在vSphere上从头开始部署Kubernetes，以及如何配置vSphere云服务提供商的细节。


### 让我们从vSphere云服务提供商开始

vSphere云服务提供商与kubernetes一起，可以允许Kubernetes pod使用企业级的vSphere存储。


### 在vSphere上部署Kubernetes

关于在vSphere上部署Kubernetes以及对vSphere云服务提供商的使用，可以参考[Kubernetes-Anywhere](https://github.com/kubernetes/kubernetes-anywhere)。

详细的步骤可以在[在vSphere上使用Kubernetes-Anywhere](https://git.k8s.io/kubernetes-anywhere/phase1/vsphere/README.md)页面上找到。


### vSphere云服务提供商

vSphere云服务提供商允许Kubernetes使用vSphere管理的企业级存储。它支持：

- 企业级服务，比如通过vSAN，QoS，高可用性和数据可靠性，实现去重和加密。
- 在容器卷粒度提供基于策略的管理
- 卷、持久卷、存储类、卷的动态配置、和使用StatefulSets为有状态的Apps提供可扩展的部署。

更多信息，请访问[vSphere Storage for Kubernetes Documentation](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/index.html).

关于如何使用vSphere管理的存储的相关文档可以在[持久卷用户指南](/docs/concepts/storage/persistent-volumes/#vsphere)和[卷用户指南](/docs/concepts/storage/persistent-volumes/#vsphere)中找到。

相关实例可以在【这里】(https://github.com/kubernetes/examples/tree/master/staging/volumes/vsphere)找到。


#### 激活vSphere云服务提供商

如果Kubernetes集群不是通过Kubernetes-Anywhere部署的，请按照下面的方法来激活vSphere云服务提供商。如果你使用的是Kubernetes-Anywhere，就无需这些步骤了，因为这些步骤在部署过程中就已经完成了。

**步骤-1** 【创建一个VM目录】(https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.vcenterhost.doc/GUID-031BDB12-D3B2-4E2D-80E6-604F304B4D0C.html)，并把Kubernetes节点VM都移到这个目录下。

**步骤-2** 确保节点VM的名字必须满足正则表达式`[a-z](([-0-9a-z]+)?[0-9a-z])?(\.[a-z0-9](([-0-9a-z]+)?[0-9a-z])?)*`。如果节点VM的名字不满足该正则表达式，则需要把它重命名，确保满足该正则表达式。

  节点VM名字的限制：
  
  * VM的名字不能以数字开头。
  * VM的名字不能有大写字母、不能有除‘.’和‘-’以外的任何特殊字符。
  * VM的名字不能少于三个字符，也不能多过63个字符。


**步骤-3** 在节点VM上激活磁盘的UUID

每个节点VM的disk.EnableUUID参数必须设置为“TRUE”。这个步骤是需要的，这样保证VMDK对VM总是呈现相同的UUID，进而允许磁盘正确的挂载。

对即将加入集群中的所有VM节点，使用[GOVC工具](https://github.com/vmware/govmomi/tree/master/govc)执行以下步骤：

* 搭建GOVC环境

		export GOVC_URL='vCenter IP OR FQDN'
        export GOVC_USERNAME='vCenter User'
        export GOVC_PASSWORD='vCenter Password'
        export GOVC_INSECURE=1

* 找出节点VM的路径

		govc ls /datacenter/vm/<vm-folder-name>
		
* 将所有VM的disk.EnableUUID设置为true

		govc vm.change -e="disk.enableUUID=1" -vm='VM 路径'
		
注意： 如果Kubernetes的节点VM都是使用模板VM创建的，在模板VM里面设置`disk.EnableUUID=1`即可。从该模板克隆出来的VM自动继承该属性。


**步骤-4** 为vSphere云服务提供商用户和vSphere实体创建并分配角色

注意： 如果你想使用管理员账号，可以略过此步。

vSphere云服务提供商要与vCenter进行交互，至少需要以下一组特权。请参考[vSphere文档中心](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.security.doc/GUID-18071E9A-EED1-4968-8D51-E0B4F526FDA3.html)了解创建客户角色、用户和角色分配的具体步骤。

<table>
<thead>
<tr>
  <th>Roles</th>
  <th>Privileges</th>
  <th>Entities</th>
  <th>Propagate to Children</th>
</tr>
</thead>
<tbody><tr>
  <td>manage-k8s-node-vms</td>
  <td>Resource.AssignVMToPool<br> System.Anonymous<br> System.Read<br> System.View<br> VirtualMachine.Config.AddExistingDisk<br> VirtualMachine.Config.AddNewDisk<br> VirtualMachine.Config.AddRemoveDevice<br> VirtualMachine.Config.RemoveDisk<br> VirtualMachine.Inventory.Create<br> VirtualMachine.Inventory.Delete</td>
  <td>Cluster,<br> Hosts,<br> VM Folder</td>
  <td>Yes</td>
</tr>
<tr>
  <td>manage-k8s-volumes</td>
  <td>Datastore.AllocateSpace<br> Datastore.FileManagement<br> System.Anonymous<br> System.Read<br> System.View</td>
  <td>Datastore</td>
  <td>No</td>
</tr>
<tr>
  <td>k8s-system-read-and-spbm-profile-view</td>
  <td>StorageProfile.View<br> System.Anonymous<br> System.Read<br> System.View</td>
  <td>vCenter</td>
  <td>No</td>
</tr>
<tr>
  <td>ReadOnly</td>
  <td>System.Anonymous<br>System.Read<br>System.View</td>
  <td>Datacenter,<br> Datastore Cluster,<br> Datastore Storage Folder</td>
  <td>No</td>
</tr>
</tbody>
</table>


**步骤-5** 创建vSphere云配置文件(`vsphere.conf`)。云配置模板可以在【这里】(https://github.com/kubernetes/kubernetes-anywhere/blob/master/phase1/vsphere/vsphere.conf)找到。

**```vsphere.conf``` for Master Node:**

```
[Global]
        user = "vCenter username for cloud provider"
        password = "password"
        server = "IP/FQDN for vCenter"
        port = "443" #Optional
        insecure-flag = "1" #set to 1 if the vCenter uses a self-signed cert
        datacenter = "Datacenter name" 
        datastore = "Datastore name" #Datastore to use for provisioning volumes using storage classes/dynamic provisioning
        working-dir = "vCenter VM folder path in which node VMs are located"
        vm-name = "VM name of the Master Node" #Optional
        vm-uuid = "UUID of the Node VM" # Optional        
[Disk]
    scsicontrollertype = pvscsi
```


注意：  **```vm-name```参数是在1.6.4发行版中引入的。** ```vm-uuid``` 和 ```vm-name```都是可选项。如果指定了```vm-name```，则```vm-uuid```会被忽略。如果两个参数都没有指定，那么kubelet会从`/sys/class/dmi/id/product_serial`中获取vm-uuid，然后查询vCenter得到节点VM的名字。

**```vsphere.conf```是工作节点的： ** （只适用于1.6.4及以上的版本。对老的版本而言，这个文件必须具有master节点```vSphere.conf```文件中指定的所有参数）。
``` 
[Global]
        vm-name = "工作节点的VM名字"
```


下面是`vsphere.conf` 文件中支持的一些参数

* ```user``` 是vSphere云服务提供商中vCenter的用户名。
* ```password``` 是vCenter用户`user`的密码。
* ```server``` 是vCenter server的IP或FQDN。
* ```port``` 是vCenter server的端口，如果没有指定，默认是443。
* ```insecure-flag``` 如果vCenter使用的是自签名的证书，则该值为1。
* ```datacenter``` 是部署节点VM的数据中心的名字。
* ```datastore``` 是使用存储类/动态配置提供卷时默认使用的datastore。
* ```vm-name``` 是最近新增的配置参数。这是一个可选参数。当这个参数出现时，工作节点上的```vsphere.conf```文件不需要vCenter的认证信息。

**注意:** ```vm-name```是版本1.6.4新增的。之前的版本不支持这个参数。


* ```working-dir``` 如果节点VM位于根VM目录下，则该值可以设置为空 ( working-dir = "")。
* ```vm-uuid``` 是VM的VM实例UUID。```vm-uuid```可以设置为空 (```vm-uuid = ""```)。 如果设置为空，可以从VM的/sys/class/dmi/id/product_serial文件中重新获得 (需要root访问权限)。

  * ```vm-uuid``` 需要设置为这种格式 - ```423D7ADC-F7A9-F629-8454-CE9615C810F1```

  * ```vm-uuid``` 可以使用如下命令从节点VM中重新获得。每个节点VM的命令不同。

        cat /sys/class/dmi/id/product_serial | sed -e 's/^VMware-//' -e 's/-/ /' | awk '{ print toupper($1$2$3$4 "-" $5$6 "-" $7$8 "-" $9$10 "-" $11$12$13$14$15$16) }'

* `datastore` 指的是使用存储类提供卷时默认的datastore。如果datastore位于存储目录下，或者datastore是某个datastore集群的一部分，请确保指定完整的datastore路径。同时还需要确保vSphere云服务提供商拥有datastore集群或用于找到datastore的存储目录的读权限。

  * 对在datastore集群中的datastore而言，按照如下方式指定datastore

        datastore = "DatastoreCluster/datastore1"

  * 对位于存储目录下的datastore而言，按照如下方式指定
  
        datastore = "DatastoreStorageFolder/datastore1"

**步骤-6** 在controller-manager, API server和 Kubelet上添加标记，进而激活vSphere云服务提供商。
* 在每个节点上运行的kubelet，controller-manager和API pod manifest文件上加上如下标记。

```
--cloud-provider=vsphere
--cloud-config=<Path of the vsphere.conf file>
```


API server和Controller-manager的manifest文件一般都在`/etc/kubernetes/manifests`下。

**步骤-7** 在所有节点上重启kubelet

* 使用```systemctl daemon-reload```重新加载kubelet systemd unit文件
* 使用```systemctl restart kubelet.service```重启kubelet服务


#### 已知问题

请访问【已知问题】(https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/known-issues.html)查看kubernetes vSphere云供应商的主要已知问题列表。


## 支持等级

要想获得快速服务，请加入 VMware Code Slack ([#kubernetes](https://vmwarecode.slack.com/messages/kubernetes/))，并发布你的问题。

IaaS Provider        | Config. Mgmt | OS     | Networking | Docs                                          | Conforms  | Support Level
-------------------- | ------------ | ------ | ---------- | --------------------------------------------- | --------- | ----------------------------
Vmware vSphere       | Kube-anywhere    | Photon OS | Flannel         | [docs](/docs/getting-started-guides/vsphere)                                |                | Community  ([@abrarshivani](https://github.com/abrarshivani)), ([@kerneltime](https://github.com/kerneltime)), ([@BaluDontu](https://github.com/BaluDontu)), ([@luomiao](https://github.com/luomiao)), ([@divyenpatel](https://github.com/divyenpatel))

如果你在使用vSphere云服务提供商时发现了任何问题，请参考【解决方法表】(/docs/getting-started-guides/#table-of-solutions)部分。
