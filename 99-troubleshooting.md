# check cluster health

## kubelet
- sudo systemctl status kubelet

## cri-dockerd 
- sudo crictl --runtime-endpoint unix:///var/run/cri-dockerd.sock ps | grep kube-apiserver

## find api ip
- cat $HOME/.kube/config | grep server