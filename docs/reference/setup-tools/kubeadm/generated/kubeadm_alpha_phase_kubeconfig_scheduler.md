---
title: 生成 kubeconfig 文件给调度器使用
approvers:
cn-approvers:
- okzhchy
---



生成 kubeconfig 文件给调度器使用

### 概要


生成 kubeconfig 文件给调度器使用且把它保存至/etc/kubernetes/scheduler.conf。

Alpha 免责声明：此命令处于 Alpha 阶段。



```
kubeadm alpha phase kubeconfig scheduler
```



### 选择项

```
      --apiserver-advertise-address string   可访问 API server 的IP地址
      --apiserver-bind-port int32            可访问 API server 的端口（默认值 6443）
      --cert-dir string                      证书存储路径（默认值 "/etc/kubernetes/pki"）
      --config string                        kubeadm 配置文件存储路径（警告: 配置文件使用是实验性的）
      --kubeconfig-dir string                kubeconfig 文件存储路径（默认值 "/etc/kubernetes"）
```
