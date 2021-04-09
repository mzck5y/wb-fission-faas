## How to install Fission Serverless Functions in an Ubuntu 20 VPS.

#### Digital Ocean Refeal Link
Use the following link to get 100 credit from Digital Ocean. If you use this refeal some credit will be added to my account

```
https://m.do.co/c/c5f51307a8a2
```

#### Install Server Requirenemtns

1. Create a non root user,
```
$ adduser faas
```

2. Add the user to sudo group
```
$ usermod -aG sudo faas
```

3. Copy the SSH key to user's home directory.
```
$ rsync --archive --chown faas:faas .ssh /home/faas
```

4. Install K3S.
```
$ curl -sfL https://get.k3s.io | sh -
```

5. Configure K3S
```
$ cd ~
$ mkdir .kube
$ sudo rsync --archive --chown faas:faas /etc/rancher/k3s/k3s.yaml ~/.kube/config
$ export KUBECONFIG=$HOME/.kube/config
```

6. Install Helm 3
```
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

7. Crate Fission Namespace
```
$ export FISSION_NAMESPACE="fission"
$ kubectl create namespace $FISSION_NAMESPACE
```


8. Install Fission on K3S, Minikube or any other cluster without LoadBalancer
```
$ helm install --namespace $FISSION_NAMESPACE --name-template fission \
    --set serviceType=NodePort,routerServiceType=NodePort,logger.enableSecurityContext=true,prometheus.enabled=false \
    https://github.com/fission/fission/releases/download/1.12.0/fission-all-1.12.0.tgz

```

9. Install Fission with Load Balancer, GCP, Azure, Amazon, DigitalOcean
```
$ helm install --namespace $FISSION_NAMESPACE --name-template fission \
    https://github.com/fission/fission/releases/download/1.12.0/fission-all-1.12.0.tgz
```


10. Install CLI
```
$ curl -Lo fission https://github.com/fission/fission/releases/download/1.12.0/fission-cli-linux && chmod +x fission && sudo mv fission /usr/local/bin/
```

11. Check the cli is installed 
```
$  fission version
```

#### Create your first function, environment and router

1. Create an environment
```
$ fission env create --name nodejs --image fission/node-env
```

2. Get a hello world sample function
```
$ curl https://raw.githubusercontent.com/fission/fission/master/examples/nodejs/hello.js > hello.js
```

3. Register the function with Fission
```
$ fission function create --name hello --env nodejs --code hello.js
```

4. Test the function
```
$ fission function test --name hello
```

5. Create an internal HTTP Trigger
```
$ fission httptrigger create --name hello-get-http-trigger-internal --url /hello --method GET --function hello
```

6. Test http trigger note that $FISSION_ROUTER is the cluster ip from the router service
```
# Get the router ip 
$ kubectl --namespace fission get svc router

# Set FISSION_ROUTER
$ export FISSION_ROUTER=ip-from-router-service

# Invoke the function
$ curl http://$FISSION_ROUTER/hello-internal
```

7. Create an HTTP Trigger with Ingress
```
$ fission httptrigger create --name hello-get-http-trigger --function hello --url /hello-ext --method GET --createingress --ingressrule "*=/hello"
```


#### Happy Codding !!!