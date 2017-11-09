# spinnaker-install
Step to install spinnaker

## Prerequisites

- Ubuntu 16.04 to host Halyard node: node_halyard
- Install kubernetes cluster: copy k8s_master:/root/.kube/config to node_halyard:/root/.kube/config
- Install Ceph cluster
endpoint: http://ceph_node:7480
access_key: DYKXYQGYQ58FCPXTOLHZ
secret_key: XXRDzs4LZsoW0Ym1a58uLjnCbMpooNKiPCnJDXOa

- Install s3 browser on Windows.
Connect to Ceph cluster and create bucket with name like spinnakerXX (spinnaker01, spinnaker02)

- Check the configurations:

### Install Halyard

Refer Docker section in official guide https://www.spinnaker.io/setup/install/halyard.

Login as root account to Halyard node.

#### Install docker

```
root@halyard01:~# apt-get update

root@halyard01:~# apt-get install \
	apt-transport-https \
	ca-certificates \
	curl \
	software-properties-common
...

root@halyard01:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK

root@halyard01:~# apt-key fingerprint 0EBFCD88
pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22

root@halyard01:~# add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

root@halyard01:~# apt-get update
root@halyard01:~# apt-get install docker-ce
```

* Verify Docker

root@halyard01:~# docker run -it ubuntu bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
ae79f2514705: Pull complete
c59d01a7e4ca: Pull complete
41ba73a9054d: Pull complete
f1bbfd495cc1: Pull complete
0c346f7223e2: Pull complete
Digest: sha256:6eb24585b1b2e7402600450d289ea0fd195cfb76893032bbbb3943e041ec8a65
Status: Downloaded newer image for ubuntu:latest


### Start Halyard

```
# docker pull gcr.io/spinnaker-marketplace/halyard:stable
# mkdir ~/.hal
# docker run -p 8084:8084 -p 9000:9000 --name halyard --rm \
	-v ~/.hal:/root/.hal \
	gcr.io/spinnaker-marketplace/halyard:stable
 _           _                     _
| |__   __ _| |_   _  __ _ _ __ __| |
| '_ \ / _` | | | | |/ _` | '__/ _` |
| | | | (_| | | |_| | (_| | | | (_| |
|_| |_|\__,_|_|\__, |\__,_|_|  \__,_|
               |___/

2017-11-08 02:53:41.811  INFO 7 --- [           main] com.netflix.spinnaker.halyard.Main       : Starting Main v0.35.0-20171002192634 on 4e2d05bb9a4b with PID 7 (/opt/halyard/lib/halyard-web-0.35.0-20171002192634.jar started by root in /workdir)
2017-11-08 02:53:41.825  INFO 7 --- [           main] com.netflix.spinnaker.halyard.Main       : The following profiles are active: test,local
2017-11-08 02:53:42.003  INFO 7 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@8317c52: startup date [Wed Nov 08 02:53:42 UTC 2017]; root of context hierarchy
2017-11-08 02:53:42.963  INFO 7 --- [           main] o.s.b.f.s.DefaultListableBeanFactory     : Overriding bean definition for bean 'retrofitLogLevel' with a different definition: replacing [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=retrofitConfiguration; factoryMethodName=retrofitLogLevel; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [com/netflix/spinnaker/config/RetrofitConfiguration.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=retrofitConfig; factoryMethodName=retrofitLogLevel; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [com/netflix/spinnaker/halyard/config/config/v1/RetrofitConfig.class]]
2017-11-08 02:53:43.104  INFO 7 --- [           main] o.s.b.f.s.DefaultListableBeanFactory     : Overriding bean definition for bean 'httpRequestHandlerAdapter' with a different definition: replacing [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration$EnableWebMvcConfiguration; factoryMethodName=httpRequestHandlerAdapter; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.data.rest.webmvc.config.RepositoryRestMvcConfiguration; factoryMethodName=httpRequestHandlerAdapter; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/data/rest/webmvc/config/RepositoryRestMvcConfiguration.class]]
...
...

### Configure Halyard

Follow below steps to configure halyard before halyard deploy.

```
docker exec -it halyard bash
```

Copy kubernetes configuration for Spinnaker to deploy to.
```
# mkdir ~/.kube
# scp 136.225.241.50:/root/.kube/config ~/.kube/
The authenticity of host '136.225.241.50 (136.225.241.50)' can't be established.
ECDSA key fingerprint is 8b:57:05:fa:5e:d3:dd:9f:2a:5e:cf:6e:42:95:0d:a3.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '136.225.241.50' (ECDSA) to the list of known hosts.
root@136.225.241.50's password:
config                                                                                                                          100% 5454     5.3KB/s   00:00
root@5de4025ac6c4:/workdir# kubectl get nodes
NAME                  STATUS    AGE       VERSION
echo                  Ready     1d        v1.8.2
node-136-225-241-51   Ready     20h       v1.8.2
node-136-225-241-52   Ready     20h       v1.8.2
node-136-225-241-53   Ready     20h       v1.8.2
node-136-225-241-54   Ready     20h       v1.8.2
```

Set Env:
```
ACCOUNT="01-spin-echo"
ADDRESS=https://armdocker.url
REPOSITORIES=proj_adp/test
USER_NAME=ciflash  ( account to login ARM server)
```

#### Config Spinnaker Environment

```
hal config version  ( to list the version)
hal config version edit --version 1.5.0

hal config provider docker-registry enable
hal config provider docker-registry account add armdocker \
    --address $ADDRESS \
    --repositories $REPOSITORIES \
    --username $USER_NAME \
    --password
hal config provider docker-registry account list


#### Config Cloud Provider

```
hal config provider kubernetes enable
hal config provider kubernetes account add $ACCOUNT --docker-registries armdocker
```

#### config Storage with Ceph

Edit file ~/.hal/config, search s3:
```
  persistentStorage:
    azs: {}
    gcs:
      rootFolder: front50
    redis: {}
    s3:
      rootFolder: front50
    oraclebmcs: {}
```
to
```
  persistentStorage:
    persistentStoreType: s3
    azs: {}
    gcs:
      rootFolder: front50
    redis: {}
    s3:
      rootFolder: front50
      region: default
      endpoint: http://ceph_node:7480
      accessKeyId: DYKXYQGYQ58FCPXTOLHZ
      secretAccessKey: XXRDzs4LZsoW0Ym1a58uLjnCbMpooNKiPCnJDXOa
      bucket: spinnaker01
    oraclebmcs: {}
```

Or follow the offical web to do if you use: 

#### Misc config
Configure LADP authentication

Edit file ~/.hal/config, search authn:
```
  security:
    ...
    authn:
      oauth2:
        enabled: false
        client: {}
        resource: {}
        userInfoMapping: {}
      saml:
        enabled: false
      ldap:
        enabled: true
        url: ldap://ldap.url:389/ou=Internal,o=xxx
        userDnPattern: uid={0},ou=users
```

#### Deploy Spinnaker to kubernetes
	
root@4e2d05bb9a4b:~/.hal# hal config deploy edit --type distributed --account-name $ACCOUNT
+ Get current deployment
  Success
+ Get the deployment environment
  Success
+ Edit the deployment environment
  Success
+ Successfully updated your deployment environment.

# hal deploy apply  
+ Get current deployment
  Success
^ Apply deployment
  Determining status of spin-orca-bootstrap: Manually deploying spin-orca-bootstrap
+ Apply deployment
  Success
+ Deploy spin-clouddriver
  Success
+ Deploy spin-front50
  Success
+ Deploy spin-orca
  Success
+ Deploy spin-deck
  Success
+ Deploy spin-echo
  Success
+ Deploy spin-gate
  Success
+ Deploy spin-igor
  Success
+ Deploy spin-rosco
  Success
Problems in default.provider.kubernetes.01-spin-echo:
- WARNING You have not specified a Kubernetes context in your
  halconfig, Spinnaker will use "kubernetes-admin@kubernetes" instead.
? We recommend explicitly setting a context in your halconfig, to
  ensure changes to your kubeconfig won't break your deployment.
? Options include:
  - kubernetes-admin@kubernetes

Problems in default.security:
- WARNING Your UI or API domain does not have override base URLs
  set even though your Spinnaker deployment is a Distributed deployment on a
  remote cloud provider. As a result, you will need to open SSH tunnels against
  that deployment to access Spinnaker.
? We recommend that you instead configure an authentication
  mechanism (OAuth2, SAML2, or x509) to make it easier to access Spinnaker
  securely, and then register the intended Domain and IP addresses that your
  publicly facing services will be using.

+ Run `hal deploy connect` to connect to Spinnaker.



[root@echo spin]# kubectl get all -n spinnaker
NAME                                 DESIRED   CURRENT   READY     AGE
rs/spin-clouddriver-bootstrap-v000   1         1         1         2m
rs/spin-clouddriver-v000             1         1         1         1m
rs/spin-deck-v000                    1         1         1         1m
rs/spin-echo-v000                    1         1         1         1m
rs/spin-front50-v000                 1         1         1         1m
rs/spin-gate-v000                    1         1         1         1m
rs/spin-igor-v000                    1         1         1         1m
rs/spin-orca-bootstrap-v000          1         1         1         2m
rs/spin-orca-v000                    1         1         1         1m
rs/spin-redis-bootstrap-v000         1         1         1         3m
rs/spin-redis-v000                   1         1         1         1m
rs/spin-rosco-v000                   1         1         1         1m
rs/spin-clouddriver-bootstrap-v000   1         1         1         2m
rs/spin-clouddriver-v000             1         1         1         1m
rs/spin-deck-v000                    1         1         1         1m
rs/spin-echo-v000                    1         1         1         1m
rs/spin-front50-v000                 1         1         1         1m
rs/spin-gate-v000                    1         1         1         1m
rs/spin-igor-v000                    1         1         1         1m
rs/spin-orca-bootstrap-v000          1         1         1         2m
rs/spin-orca-v000                    1         1         1         1m
rs/spin-redis-bootstrap-v000         1         1         1         3m
rs/spin-redis-v000                   1         1         1         1m
rs/spin-rosco-v000                   1         1         1         1m

NAME                                       READY     STATUS    RESTARTS   AGE
po/spin-clouddriver-bootstrap-v000-nd4fl   1/1       Running   0          2m
po/spin-clouddriver-v000-hrfrt             1/1       Running   0          1m
po/spin-deck-v000-dwnbp                    1/1       Running   0          1m
po/spin-echo-v000-5td6p                    1/1       Running   0          1m
po/spin-front50-v000-45vvd                 1/1       Running   0          1m
po/spin-gate-v000-v7lxz                    1/1       Running   0          1m
po/spin-igor-v000-9jsb2                    1/1       Running   0          1m
po/spin-orca-bootstrap-v000-nzkck          1/1       Running   0          2m
po/spin-orca-v000-mbtpn                    1/1       Running   1          1m
po/spin-redis-bootstrap-v000-smjnj         1/1       Running   0          3m
po/spin-redis-v000-zmq52                   1/1       Running   0          1m
po/spin-rosco-v000-srblh                   1/1       Running   0          1m

NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
svc/spin-clouddriver             ClusterIP   10.108.37.74     <none>        7002/TCP   1m
svc/spin-clouddriver-bootstrap   ClusterIP   10.105.126.140   <none>        7002/TCP   2m
svc/spin-deck                    ClusterIP   10.108.2.232     <none>        9000/TCP   1m
svc/spin-echo                    ClusterIP   10.111.25.15     <none>        8089/TCP   1m
svc/spin-front50                 ClusterIP   10.111.237.84    <none>        8080/TCP   1m
svc/spin-gate                    ClusterIP   10.100.206.13    <none>        8084/TCP   1m
svc/spin-igor                    ClusterIP   10.107.84.164    <none>        8088/TCP   1m
svc/spin-orca                    ClusterIP   10.107.231.33    <none>        8083/TCP   1m
svc/spin-orca-bootstrap          ClusterIP   10.100.172.155   <none>        8083/TCP   2m
svc/spin-redis                   ClusterIP   10.104.97.87     <none>        6379/TCP   1m
svc/spin-redis-bootstrap         ClusterIP   10.107.135.42    <none>        6379/TCP   3m
svc/spin-rosco                   ClusterIP   10.102.59.137    <none>        8087/TCP   1m


### Update installation 

* create ingress for deck and gate services

root@97917a8b7d87:/workdir# hal config security ui edit --override-base-url http://spinnaker.echo.seli.gic.ericsson.se
+ Get current deployment
  Success
+ Get UI security settings
  Success
+ Edit UI security settings
  Success
Problems in default.security:
- WARNING Your UI or API domain does not have override base URLs
  set even though your Spinnaker deployment is a Distributed deployment on a
  remote cloud provider. As a result, you will need to open SSH tunnels against
  that deployment to access Spinnaker.
? We recommend that you instead configure an authentication
  mechanism (OAuth2, SAML2, or x509) to make it easier to access Spinnaker
  securely, and then register the intended Domain and IP addresses that your
  publicly facing services will be using.
