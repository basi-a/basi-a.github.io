---
title: 用Ansible解放搞k8s集群的双手
date: 2024-01-23 22:17:18
updated: 2024-01-23 22:17:18
tags: [kubernetes,Ansible]
categories: kubernetes
keywords: 
description:
top_img:
comments:
cover:
toc:
toc_number:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
---
# 前言

> 之前手装过k8s，这次想着尽可能自动化，但全自己手搓脚本有点累，而且有现成的我为啥还要重复造轮子呢（才不是嫌麻烦不想写了）。
>
> 于是呢，就有了下面这些东西，掏出了`Ansible`, 说实话，要不是之前面试我还真不知道有`playbook`这东西。

# k8s初始环境

## 基础环境列表

`hosts.k8s`

```text
192.168.122.10	master
192.168.122.11	node01
192.168.122.12	node02
192.168.122.13	node03
```
这些全都装的`ubuntu 22.04`
## 免密认证,同步hosts,设定主机名

```bash
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
cat hosts.k8s | sudo tee -a /etc/hosts
```

为了舒服，咱就别干撸shell了，上`ansible`

```bash
sudo apt-get -y sshpass ansible
```

创建主机清单`inventory.ini`：

```bash
[master]
master ansible_host=master ansible_user=basi
[nodes]
node01 ansible_host=node01 ansible_user=basi
node02 ansible_host=node02 ansible_user=basi
node03 ansible_host=node03 ansible_user=basi
```

`palybook.yml`内容如下：

```yml
---
- hosts: all
  tasks:
    - name: Set authorized key taken from file
      authorized_key:
        user: basi
        state: present
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

    - name: Copy the hosts file
      become: yes
      copy:
        src: /etc/hosts
        dest: /etc/hosts
      changed_when: false

    - name: Get the hostname from the hosts file
      shell: "awk '/{{ ansible_default_ipv4.address }}/ {print $2}' /etc/hosts"
      register: new_hostname
      changed_when: false

    - name: Update the hostname
      become: yes
      hostname:
        name: "{{ new_hostname.stdout }}"
      changed_when: false

    - name: Get the Changed hostname
      shell: "cat /proc/sys/kernel/hostname"
      register: result
      changed_when: false

    - name: Output the Changed hostname
      debug:
        var: result.stdout_lines
      changed_when: false

```

`playbook` 开跑！！！

```bash
ansible-playbook -i inventory.ini playbook.yml -k -K
# 先输入一下用户密码，再输入一下sudo密码，搞定
```

![ansible-playbook.png](https://cdn.basi-a.top/images/ansible-playbook.png)

# 安装contrainerd, 通用设置

## 安装containerd

先手动下载一下[安装包](https://github.com/containerd/containerd/releases/latest)，咱选`cri-containerd-cni`这个啥都有的, 看好架构哈, 我这个是当前的最新版本1.6.27

```bash
wget -c -v https://github.com/containerd/containerd/releases/download/v1.6.27/cri-containerd-cni-1.6.27-linux-amd64.tar.gz
```

`containerd_playbook.yml` 内容如下

```yaml
---
- hosts: all
  tasks:
    - name: Copy the tarball
      become: yes
      copy:
        src: ~/cri-containerd-cni-1.6.27-linux-amd64.tar.gz
        dest: /opt/cri-containerd-cni-1.6.27-linux-amd64.tar.gz
      changed_when: false
    - name: Install cri-containerd-cni with tarball
      become: yes
      shell: "tar -zxvf /opt/cri-containerd-cni-1.6.27-linux-amd64.tar.gz -C /"
      changed_when: false
    - name: Change Cgroup to systemd and start conatainerd
      become: yes
      shell:
      	cmd: |
      		mkdir /etc/containerd
      		containerd config default | tee /etc/containerd/config.toml
      		sed -i '/\[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options\]/,/SystemdCgroup/ s/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
      		systemctl enable containerd --now
      changed_when: false
    - name: Check containerd version
      shell: "containerd --version"
      register: version
      changed_when: false
    - name: Info containerd version
      debug:
        var: version.stdout_lines
      changed_when: false
```

`playbook` 开跑！！！

```bash
ansible-playbook -i inventory.ini containerd_playbook.yml -K
```

![containerd-install.png](https://cdn.basi-a.top/images/containerd-install.png)

## 通用配置

`common_conf_set.sh`配置设置脚本内容如下

```bash
#!/bin/bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 应用 sysctl 参数而不重新启动
sysctl --system
systemctl disable ufw --now
```

`common_playbook.yml`内容如下

```yaml
---
- hosts: all
  tasks:
    - name: Copy the common setting script
      become: yes
      copy:
        src: ~/common_conf_set.sh
        dest: /opt/common_conf_set.sh
      changed_when: false
    - name: Run script
      become: yes
      shell: "/bin/bash /opt/common_conf_set.sh"
      changed_when: false
    - name: Check config
      shell: "sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward"
      register: result
      changed_when: false
    - name: Info config
      debug:
        var: result.stdout_lines
      changed_when: false
```

`playbook` 开跑！！！

```bash
ansible-playbook -i inventory.ini common_playbook.yml -K
```

![common_comfig](https://cdn.basi-a.top/images/common_config.png)

# 安装kubeadm, kubelet, kubectl

## 导入软件源，并且安装组件

`k8s_apt_add.sh`  我在写这个的时候最新版本就是v1.29

```bash
apt-get update && apt-get install -y apt-transport-https
curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.29/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.29/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
sleep 5
```

`k8s_apt_palybook.yml`

```yaml
---
- hosts: all
  tasks:
    - name: Add k8s to apt keying and install kubelet kubeadm kubectl
      become: yes
      copy:
        src: ~/k8s_apt_add.sh
        dest: /opt/k8s_apt_add.sh
      changed_when: false
    - name: Run script
      become: yes
      shell: "/bin/bash /opt/k8s_apt_add.sh"
      changed_when: false
    - name: Check version
      shell: "kubeadm version && echo \"kubelet version: $(kubelet --version)\" && echo \"kubectl version: $(kubectl version)\""
      register: result
      changed_when: false
    - name: Info version
      debug:
        var: result.stdout_lines
      changed_when: false
```

`playbook` 开跑！！！

```bash
ansible-playbook -i inventory.ini k8s_apt_playbook.yml -K
```

![kubeadm-kubelet-kubectl-install](https://cdn.basi-a.top/images/kubeadm-kubelet-kubectl-install.png)

# 安装k8s

## 禁用swap

写了个swap关闭脚本`swapoff.sh`

```bash
#!/bin/bash
swapoff -a && sed -i 's/^\([^#]*swap.*\)/#&/' /etc/fstab
if [ "$(cat /etc/swaps | awk '{if(NR>1){print $0}}')" == "" ];then
echo "Swap has been shut down !!!"
else
echo "Swap is still in place now !!!"
fi
```

`swapoff_playbook.yml` 内容如下

```yaml
---
- hosts: all
  tasks:
    - name: Copy script
      become: yes
      copy:
        src: ~/swapoff.sh
        dest: /opt/swapoff.sh
      changed_when: false
    - name: Turn off and check
      become: yes
      shell: "/bin/bash /opt/swapoff.sh"
      register: result
      changed_when: false
    - name: Info check result
      debug:
        var: result.stdout_lines
      changed_when: false
```

`playbook` 开跑！！！

```bash
ansible-playbook -i inventory.ini swapoff_playbook.yml -K
```

![turnoffswap](https://cdn.basi-a.top/images/turnoffswap.png)

## 开装

`master_playbook.yml`

```yaml
- hosts: master
  tasks:
    - name: initialize the cluster
      become: yes
      shell: "kubeadm init --control-plane-endpoint=master --pod-network-cidr=10.244.0.0/16 && sleep 5"
      changed_when: false
    - name: create .kube directory
      become: yes
      become_user: basi
      file:
        path: /home/basi/.kube
        state: directory
        mode: '0755'
      changed_when: false
    - name: copy admin.conf
      become: yes
      copy:
        remote_src: yes
        src: /etc/kubernetes/admin.conf
        dest: /home/basi/.kube/config
        group: basi
        owner: basi
      changed_when: false
    - name: Install Cni flannel
      shell: "kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml && sleep 5"
      changed_when: false
    - name: Get join token
      become: yes
      shell: "kubeadm token create --print-join-command"
      register: kubernetes_join_command
      changed_when: false
    - name: Copy join command to local file.
      become: yes
      local_action: copy content="{{ kubernetes_join_command.stdout_lines[0] }}" dest="/tmp/kubernetes_join_command" mode=0755
      changed_when: false
    - name: Check Master
      shell: "kubectl get nodes"
      register: result
      changed_when: false
    - name: Info Master status
      debug:
        var: result.stdout_lines

```

`nodes_playbook.yml` 

```yaml
- hosts: nodes
  become: yes
  tasks:
    - name: Check and remove /etc/kubernetes/kubelet.conf if it exists
      file:
        path: /etc/kubernetes/kubelet.conf
        state: absent
      changed_when: false

    - name: Check and remove /etc/kubernetes/pki/ca.crt if it exists
      file:
        path: /etc/kubernetes/pki/ca.crt
        state: absent
      changed_when: false

    - name: Copy join command from Ansiblehost to the worker nodes.
      copy:
        src: /tmp/kubernetes_join_command
        dest: /tmp/kubernetes_join_command
        mode: '0755'
      changed_when: false

    - name: Join the Worker nodes to the cluster.
      command: "/bin/bash /tmp/kubernetes_join_command"
      register: joined_or_not
      changed_when: false

    - name: Info result
      debug:
        var: joined_or_not.stdout_lines
      changed_when: false                               
```

`playbook` 开跑！！！

```bash
ansible-playbook -i inventory.ini master_playbook.yml -K
ansible-playbook -i inventory.ini nodes_playbook.yml -K
```

![master](https://cdn.basi-a.top/images/k8s-master.png)

![nodes-join](https://cdn.basi-a.top/images/k8s-nodes-join.png)

# k8s 启动！！！！

## 先改一下`<none>`的角色名, 顺便看看状态

```bash
kubectl label node node01 node-role.kubernetes.io/worker=worker
kubectl label node node02 node-role.kubernetes.io/worker=worker
kubectl label node node03 node-role.kubernetes.io/worker=worker
```

![roles](https://cdn.basi-a.top/images/roles.png)

## 装个管理面板

到github上找一下最新版本的[`dashboard`](https://github.com/kubernetes/dashboard/releases/latest),  我现在的主线版本是`v3.0.0-alpha0`, 从这个版本起底层架构发生了变化。

> Starting from the release of the Kubernetes Dashboard, the underlying architecture has changed, and it requires a clean installation. Please remove the previous installation first.`v3`
>
> Kubernetes Dashboard now uses [cert-manager](https://cert-manager.io/docs/installation/) and [nginx-ingress-controller](https://docs.nginx.com/nginx-ingress-controller/installation/) by default to work properly. Please make sure you have them installed in your cluster if you want to use a manifest-based installation path. The helm-based approach can install all required dependencies automatically for you if needed.

开装，得先安装一下[`cert-manager`](https://cert-manager.io/docs/installation/)和[`nginx-ingress-controller`](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v3.0.0-alpha0/charts/kubernetes-dashboard.yaml
```

有点尴尬呢，这个版本还没适配1.29的k8s，先这样，等更新

# 结语

> 咋说呢.....
>
> 感觉我这玩意有很大的优化空间，先这样，就当练手了，哪天再细想去.......
>
> 得搞个`harbor`做私有仓库
>
> 哪天在添个加外部`etcd`集群的，看看在弄个双`master`的，，再上个用`keepalived`做主备的双`haproxy`给这俩`master`做均衡负载