# k3s-ssh-service-demo

## Summary

A demo of a k3s cluster running pods accessible via ssh and with distinct IPs or names. This code is derived from an [example](https://gist.github.com/jpetazzo/63ad363937ce5b7d48ed4af8e06fe38b) provided by [Jérôme Petazzoni](https://jpetazzo.github.io/). 

## Basic Setup of k3s

Install a brand new k3s on the target machine using the curl method

```bash
curl -sfL https://get.k3s.io | sh - 
```

## Apply the settings and run the service

Either apply all configuration files at once via calling it for the directory `config`

```bash
kubectl apply -f config
```

or apply the settings file by file by issuing the following commands:

```bash
sudo kubectl apply -f pvc.yaml 
sudo kubectl apply -f configmap.yaml 
sudo kubectl apply -f service.yaml 
sudo kubectl apply -f pod.yaml 
```

## Check for Success

You can check the success by running

```bash
sudo kubectl get pod shpod
sudo kubectl describe configmaps shpod
```

an log into the container as root via

```bash
sudo kubectl exec -it shpod -- /bin/sh
```

If logged in, you get a prompt and can see that the mounting points are available

```bash
/ # cat /home/user/.ssh/authorized_keys 
ssh-ed25519 AAAAC3NzaC1lZDI1NTEXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX+qzZ numer@pc-001
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr F6:BB:D9:DC:BE:4B  
          inet addr:10.42.0.10  Bcast:10.42.0.255  Mask:255.255.255.0
          inet6 addr: fe80::f4bb:d9ff:fedc:be4b/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:924 errors:0 dropped:0 overruns:0 frame:0
          TX packets:636 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:4541612 (4.3 MiB)  TX bytes:45948 (44.8 KiB)

# ... lo skipped ...
/ # ps -ef | grep ssh
    1 root      0:00 sshd: /usr/sbin/sshd -D -e [listener] 0 of 10-100 startups
```

## Setting up port forwarding

If you only want to connect from the host, it suffices to execute 

```bash
$ sudo kubectl port-forward service/shpod 2222:22
Forwarding from 127.0.0.1:2222 -> 22
Forwarding from [::1]:2222 -> 22
``` 

To connect from another machine in the network, one must use a more permissive filter for port forwarding:

```bash
sudo kubectl port-forward --address 0.0.0.0 service/shpod 2222:22
```

## Use a dedicated IP address

[Former versions of `service.yaml`](https://github.com/daixtrose/k3s-ssh-service-demo/commit/fc1bfdf884cff647f7cc0c2438a7746ba1b90e45) contained `type: LoadBalancer` for the pod to obtain an external IP address. 
This was later changed to using `externalIPs` directly. You can query the IP via  

```bash
sudo kubectl describe services shpod
```

## Starting and Stopping the Service

While doing changes or experiments to the service configuration, you can turn off and on the service using the following commands:

```bash
sudo kubectl delete services shpod
sudo kubectl apply -f config/service.yaml 
```
