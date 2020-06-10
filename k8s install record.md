--- Centos7.5系统安装kubernetesv1.18.3记录【无需翻墙】
0.安装docker19.03.8版本
安装完成docker version查看版本

1.关闭&&禁用防火墙
systemctl stop firewalld && systemctl disable firewalld

2.关闭模式
vim /etc/selinux/config
SELINUX=enforcing改为SELINUX=disabled，保存后退出

3.集群桥连网络配置
cat /etc/sysctl.d/k8s.conf 
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
sysctl --system   ---加载配置

4.docker与kubeadmin的cgroup保持一致
 vim /etc/docker/daemon.json
 {
    "exec-opts": ["native.cgroupdriver=systemd"]
}
systemctl daemon-reload && systemctl enable docker && systemctl restart docker

5.关闭交换区
swapoff -a

6.配置国内源
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

7.拉取所需要的镜像组件(bash image.list)其中组件版本号需要通过kubeadm config images list --kubernetes-version=v1.18.2获取
#!/bin/bash
    # download k8s 1.18.2 images
    # get image-list by 'kubeadm config images list --kubernetes-version=v1.18.2'
images=(
kube-apiserver:v1.18.2
kube-controller-manager:v1.18.2
kube-scheduler:v1.18.2
kube-proxy:v1.18.2
pause:3.2
etcd:3.4.3-0
coredns:1.6.7
)
for imageName in ${images[@]};do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName  
    docker tag  registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName  
    docker rmi  registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done

8.初始化集群（init具体参数可以参考官方文档）
kubeadm init --kubernetes-version=v1.18.2   --pod-network-cidr=172.16.0.0/16 --apiserver-advertise-address=10.8.80.75

8.1如果搭建的集群需要执行此命令加入集群
kubeadm join 10.8.80.75:6443 --token 9auk33.0bzuofursqp7cwzp \
    --discovery-token-ca-cert-hash sha256:39dfc554fbdacdc551c8eb5ca6e34feca952e676203248b3567aca46832ff2a4

9.设置授权
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

10.master设置为单节点
kubectl taint nodes --all node-role.kubernetes.io/master-

11.安装calico网络插件
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml
