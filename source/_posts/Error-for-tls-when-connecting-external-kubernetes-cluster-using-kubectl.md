---
layout: post
title: kubectl连接外部k8s集群tls证书报错
date: 2023-10-20 23:06:59
tags: [kubectl, k8s, kubernetes, tls]
thumbnail: Error-for-tls-when-connecting-external-kubernetes-cluster-using-kubectl/kubectl.jpeg
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Error-for-tls-when-connecting-external-kubernetes-cluster-using-kubectl.md)

报错如下:

```bash
$ kubectl get po -A
E1001 23:16:16.888813   15102 memcache.go:265] couldn't get current server API group list: Get "https://ubuntu-w510.local:16443/api?timeout=32s": tls: failed to verify certificate: x509: certificate is valid for kubernetes, kubernetes.default, kubernetes.default.svc, kubernetes.default.svc.cluster, kubernetes.default.svc.cluster.local, not ubuntu-w510.local
```

stackoverflow上《[kubectl unable to connect to server: x509: certificate signed by unknown authority](https://stackoverflow.com/questions/46234295/kubectl-unable-to-connect-to-server-x509-certificate-signed-by-unknown-authori)》和《[helm: x509: certificate signed by unknown authority](https://stackoverflow.com/questions/48119650/helm-x509-certificate-signed-by-unknown-authority)》，介绍了以下几种办法：

```bash
# extract <-----BEGIN/END CERTIFICATE-----> from openssl to ~/.kube/myCert.crt
$ openssl s_client -showcerts -connect IP:PORT

# replace line `certificate-authority-data` to certificate-authority: myCert.crt
```

```bash
$ kubectl --insecure-skip-tls-verify ...
```

```bash
$ cat ~/.kube/insecure-skip-tls-verify.sample.yaml
clusters:
- cluster:
    server: https://cluster.mysite.com
    insecure-skip-tls-verify: true
  name: default
```

---

参考资料：
- [kubectl unable to connect to server: x509: certificate signed by unknown authority](https://stackoverflow.com/questions/46234295/kubectl-unable-to-connect-to-server-x509-certificate-signed-by-unknown-authori)
- [helm: x509: certificate signed by unknown authority](https://stackoverflow.com/questions/48119650/helm-x509-certificate-signed-by-unknown-authority)
- [Is there a way to access microk8s using kubectl outside the local network](https://discuss.kubernetes.io/t/is-there-a-way-to-access-microk8s-using-kubectl-outside-the-local-network/15488)
- [Unable to connect to the server: x509: certificate signed by unknown authority #823](https://github.com/kubernetes/kubectl/issues/823)
- [Cannot connect to microk8s from another machine #421](https://github.com/canonical/microk8s/issues/421) → [method_possible](https://github.com/canonical/microk8s/issues/421#issuecomment-599815155)

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](hello-world/donate-me.png)
