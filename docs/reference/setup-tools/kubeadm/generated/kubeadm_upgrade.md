
用此命令能将您的集群平滑升级到新版本。

### 概要


用此命令能将您的集群平滑升级到新版本。

```
kubeadm upgrade
```




 ### 选择项

```
      --allow-experimental-upgrades        显示Kubernetes的不稳定版本作为升级可选项并允许升级到Kubernetes的 alpha/beta/release 候选版本。
      --allow-release-candidate-upgrades   显示Kubernetes的发布版本作为升级可选项并允许升级至Kubernetes的发布候选版本。
      --config string                      kubeadm config 文件路径（警告：配置文件的使用是实验性的）
      --feature-gates string               一系列描述不同功能选项的feature gates的键值对：
CoreDNS=true|false (ALPHA - default=false)
DynamicKubeletConfig=true|false (ALPHA - default=false)
HighAvailability=true|false (ALPHA - default=false)
SelfHosting=true|false (BETA - default=false)
StoreCertsInSecrets=true|false (ALPHA - default=false)
SupportIPVSProxyMode=true|false (ALPHA - default=false)
      --ignore-checks-errors stringSlice   检查的错误显示成的警告列表。比如: 'IsPrivilegedUser,Swap'. 值 'all' 所有检查的错误。
      --kubeconfig string                  与集群对话时用到的 KubeConfig 文件（默认值 "/etc/kubernetes/admin.conf"）
      --print-config                       指定是否打印升级用到的配置文件。
```
