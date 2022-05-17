# k3s

Automated setup of a k3s cluster with 1 master and n compute-nodes.

## Warning
I only tested this on my local cluster running on 6 ODROID-MC1 Solo. I run them with Armbian buster (Linux 5.4.28-odroidxu4)
So use this at your own risk!

## Setup
```bash
$ cd setup
$ # update inventory to your needs
$ ansible-playbook -i inventory.yaml prep.yaml 
$ ansible-playbook -i inventory.yaml create_cluster.yaml
```

To connect to the cluster, copy the kube-config from the k3s-master to your local machine.  
```bash
# assuming you have passwordless sudo, which you're gonna need for ansible either way
$ ssh user@master "sudo cp /etc/rancher/k3s/k3s.yaml /home/<user>/k3s.yaml && sudo chown <user>:<group> /home/<user>/k3s.yaml"
$ scp <user>@<master>:k3s.yaml .
$ ssh user@master "rm -f /home/<user>/k3s.yaml"
```
  
Update k3s.yaml by replacing `server: https://127.0.0.1:6443` with the actual servername `server: https://<mastername or -ip>:6443`
Afterwards you should be able to connect to your cluster, using k3s.yaml kube-config.  
You can test it with:
```bash
$ kubectl --kubeconfig=k3s.yaml get nodes
```

## Test the functionality
In `deploy` you can find a directory called `test`, which contains a simple deployment for nginx with a service and an ingress. You can deploy it, to see if your cluster is working properly.  
```bash
$ kubectl apply -f deploy/test/
# Verify the deployment
$ kubectl get all -n test
# You should get an output similar to this:
NAME                              READY   STATUS    RESTARTS   AGE
pod/nginx-test-6ddd567766-9vdm5   1/1     Running   0          103m

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/nginx-test   ClusterIP   10.43.25.181   <none>        80/TCP    103m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-test   1/1     1            1           103m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-test-6ddd567766   1         1         1       103m
```
  
If this worked, update `/etc/hosts` to make `example.com` point to the IP of your master.
Now open http://example.com in your Browser and check, if you can see the default NGINX-homepage.
Alternatively you can test from the cli using this command  
```bash
$ curl example.com 2>/dev/null |grep "Welcome to nginx\!" || echo "Failed"
# Good
<title>Welcome to nginx!</title>
<h1>Welcome to nginx!</h1>
# Bad
Failed
```

Now you're good to go. Have fun with your k3s-cluster!
