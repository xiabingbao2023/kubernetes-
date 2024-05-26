# kubernetes的node节点修改hostname

- node节点修改hostname

  `sudo hostnamectl hostname work1`

- master节点删除该节点

  `kubectl delete nodes vm-16-10-ubuntu`

- node节点的删除相应配置信息

  ```
  #使用kubeadm重置
  sudo kubeadm reset
  #停止kubelet
  sudo systemctl stop kubelet
  #删除相应配置文件
  sudo rm -rf $HOME/.kube/
  sudo rm -rf /var/lib/cni/ /var/lib/kubelet/* /etc/cni/
  #删除网络信息
  sudo ifconfig cni0 down
  sudo ifconfig flannel.1 down
  sudo ip link delete flannel.1
  sudo ip link delete cni0
  #启动kubelet
  sudo systemctl start kubelet
  #重新加入master
  sudo kubeadm join 10.0.20.17:6443 --token intrib.orje4sbnzow0juvn     --discovery-token-ca-cert-hash sha256:99042cb26719f3eac671744792f3ae428aeea7a824ebeb5ddf5a17954dd89779
  
  ```
  
  