# Radiant on K8S connect using vscode

These are quick steps for POC of running radiant in kubernetes and connecting to it from VS Code and a browser on local machine

## Prerequisite

A running Kubernetes on installation e.g. microK8S on ubuntu
- https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#1-overview

kubectl cli utility on your laptop
- https://kubernetes.io/docs/tasks/tools/
## Connect to remote Kubernetes from your laptop

Step 1. SSH into ubuntu server and get kubeconfig file
```bash

ssh  yourusername@<SERVER NAME/IP>

# ensure kubernetes is running 

microk8s.kubectl get nodes
#NAME    STATUS   ROLES    AGE     VERSION
#libra   Ready    <none>   4h48m   v1.27.5

# fetch kubeconfig file
microk8s.kubectl config view --raw > ~/kube.config
```
Step 2. Replace the server address `https://127.0.0.1:16443` with `https://<SERVER NAME/IP>:16443` in the ~/kube.config file

Step 3. Download this file to your laptop

```bash
scp yourusername@<SERVER NAME/IP>:~/.kube.config  $HOME/kube.config
```

Step 4. Connect to Kubernetes from your laptop
```bash 

# Verify this works 
KUBECONFIG=$HOME/kube.config kubectl  --insecure-skip-tls-verify get node
#NAME    STATUS   ROLES    AGE     VERSION
#libra   Ready    <none>   4h56m   v1.27.5

```


## Start Radiant

```bash
KUBECONFIG=$HOME/kube.config kubectl  --insecure-skip-tls-verify create deployment --image vnijs/radiant radiant
# .. you should see
#deployment.apps/radiant created

KUBECONFIG=$HOME/kube.config kubectl --insecure-skip-tls-verify get pods 
#.. you should see
#NAME                      READY   STATUS              RESTARTS   AGE
#radiant-8c7fbb8d7-9h2j2   0/1     ContainerCreating   0          86sKUBECONFIG=$HOME/kube.config kubectl get pods

#.. wait 1-2 minutes again repeat above command until you see STATUS running
#NAME                      READY   STATUS    RESTARTS   AGE
#radiant-8c7fbb8d7-9h2j2   1/1     Running   0          2m59s

```

## COPY your public ssh key to the container 

```bash

cat cat ~/.ssh/id_rsa.pub
# You should see the content of your public key .. we will copy this into the ~/.ssh/authorized_keys file 
# of the container
#ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDexZgT4SZzBmB3kSlY0eYPmjw6uU1oJigEUyAIkm3/nvO2SPhWSjX4zt6ITSBA3JJUjU+eVB5JLvvaK/fvA94oyy0+T+MvFvfbUFzhakd8qPAmSBE+WkJF8ps2V...




# Note explicit POD Name you need to copy from previous output
KUBECONFIG=$HOME/kube.config kubectl --insecure-skip-tls-verify exec -it radiant-8c7fbb8d7-9h2j2 -- bash

# .. you should see
# jovyan@radiant-8c7fbb8d7-9h2j2:~$ ..

# Make dir ~/.ssh
mkdir ~/.ssh
# Copy your 
vi ~/.ssh/authorized_keys
# Copy the content from your local id_rsa.pub to here
# save file.
exit
```

## Port forward to ssh server using kubectl 
```bash
KUBECONFIG=$HOME/kube.config kubectl --insecure-skip-tls-verify port-forward radiant-8c7fbb8d7-9h2j2 11022:22 
# you should see ...
#Forwarding from 127.0.0.1:11022 -> 22
## KEEP THIS terminal running
```
## Verify that you can connect to radiant container using ssh
```bash

ssh jovyan@localhost -p 11022
# .. You should see something like 
# Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-163-generic x86_64)
# ...

 ```
If you have reached so far things are looking great.

## Connect to vs code

Now open VS Code and connect to remote ssh server using instructions at 
- https://code.visualstudio.com/docs/remote/ssh

- Add new server name=`radiant`   connection = `ssh jovyan@localhost -p 11022`

Then open a folder on remote server `radiant`

If you are able to open the remote workspace lets start radiant

## Start radiant in VS Code

In VScode right click on any file and then > `Open in integrated terminal`
This will start a ssh shell within VSCode

```bash
# Run R
jovyan@radiant-8c7fbb8d7-9h2j2:~$ R
#..
#..
#..
> radiant::radiant()

# You should see a radiant browser window pop up 
```

If all of this works out we are very lucky
