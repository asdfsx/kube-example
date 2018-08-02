# kube-example
使用kubectl启动第一个pod

```sh
kubectl create -f ./simple_pod.yaml
kubectl get pods
kubectl logs pods/myapp-pod
```

使用kubectl启动第二个pod

```sh
kubectl create -f ./liveness_pod.yaml
kubectl get pods
kubectl logs pods/liveness-http
```

第二个pod使用的镜像源码地址：[源码](https://github.com/kubernetes/kubernetes/tree/96ec3187180b9c1d722756b3ea0984ebe65424dc/test/images/liveness)

使用kubectl启动第三个pod

```sh
kubectl create -f ./initcontainer_pod.yaml
kubectl get -f ./initcontainer_pod.yaml
kubectl describe -f ./initcontainer_pod.yaml
kubectl logs pod/myapp-withinit-pod -c myapp-withinit-container
kubectl logs pod/myapp-withinit-pod -c init-mydb
kubectl logs pod/myapp-withinit-pod -c init-myservice
```

启动service

```sh
kubectl create -f ./services.yaml
```

使用kubectl启动replicaset controller

```sh
kubectl create -f ./replicaset_controller.yaml
kubectl get -f ./replicaset_controller.yaml
kubectl describe -f ./replicaset_controller.yaml
kubectl delete -f ./replicaset_controller.yaml (删除 replicaset 和 pods)
kubectl delete -f ./replicaset_controller.yaml --cascade=false (只删除 replicaset ，不删除 pods)
```

使用kubectl启动deployment controller

```sh
kubectl create -f ./deployment_controller.yaml
kubectl get deployments
kubectl get -f ./deployment_controller.yaml
kubectl describe -f ./deployment_controller.yaml
kubectl rollout status deployment/nginx-deployment
kubectl get pods --show-labels
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1   (滚动升级)
```

做个失败的滚动升级

```sh
kubectl set image deployment/nginx-deployment nginx=nginx:1.91
kubectl rollout status deployments nginx-deployment
kubectl get rs
kubectl get pods
kubectl describe deployment
kubectl rollout history deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment --revision=2
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

注意所有的命令需要加上 --record=true 参数才能在 history 里显示出 change-cause

设置autoscale

```sh
kubectl scale deployment nginx-deployment --replicas=10
kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80
```

暂停update

```sh
kubectl rollout pause deployment/nginx-deployment
kubectl set image deploy/nginx-deployment nginx=nginx:1.9.1
kubectl rollout history deploy/nginx-deployment
kubectl set resources deployment nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
kubectl rollout resume deploy/nginx-deployment
kubectl get rs -w
```

StatefrulSet
无法在本地测试，因为无法提供storageclass

使用Job

```sh
kubectl create -f ./job_controller.yaml
kbuectl get pods
pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath={.items..metadata.name})
kubectl logs $pods
```

使用cronjob

```sh
kubectl create -f cronjob_controller.yaml 
kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
kubectl get cronjob
kubectl get jobs --watch
kubectl get pods
kubectl logs hello-1533181680-rcdgm
kubectl delete cronjob -f cronjob_controller.yaml
```