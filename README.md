# cert-boot
[cert-boot](https://github.com/tsuyo/cert-boot) helm chart sources as well as helm repo (https://tsuyo.github.io/cert-boot) via GitHub Pages

## Why?
[cert-manager](https://cert-manager.io/) is a defact certificate management tool in kubernetes.
However, it would be a bit tricky to use at first (if you like me) mainly because it requires a lot of configurations in advance.
cert-boot is intended to make it easy to get started with cert-manager.

## How?
cert-boot extracts the basic settings of cert-manager and uses them as templates for the helm chart, so that users can start with minimal settings.

## Limits
Amazon Route 53 with Let's Encrypt configuration is currently supported.

# For Users
Prerequisites
1. Ingress LB IP
    - (e.g.) 34.146.46.63
2. DNS names (which you want to use as https://) associated with 1.
    - (e.g.) tsuyo.org
3. Amazon Route 53 records for the DNS names
    - (e.g.) platform-dev.tsuyo.org / A / Simple / 34.146.46.63
    - (e.g.) *.platform-dev.tsuyo.org / A / Simple / 34.146.46.63

Install cert-manager (if you don't)
```
$ helm repo add jetstack https://charts.jetstack.io # for cert-manager
$ helm upgrade cert-manager -i --create-namespace -n cert-manager --version v1.6.0 --set installCRDs=true jetstack/cert-manager
```
Install cert-boot
```
$ helm repo add cert-boot https://tsuyo.github.io/cert-boot
$ vi values.yaml
```
```
# Email for Let's Encrypt
letsencrypt:
    email: me@tsuyo.org
# Region in Amazon Route 53
route53:
    region: ap-northeast-1
# DNS names associated with Ingress LB IP
dnsNames:
    - tsuyo.org
    - platform-dev.tsuyo.org
    - '*.platform-dev.tsuyo.org'
# AWS keys
aws:
    access_key_id: xxxxxxxxxxxxxxxxxxxx
    secret_access_key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# Namespace of cert-manager
cm:
    namespace: cert-manager
# Ingress LB IP
ingress-nginx:
    controller:
        service:
            loadBalancerIP: 34.146.46.63
```
```            
$ helm upgrade cert-boot -i --create-namespace -n cert-boot -f values.yaml cert-boot/cert-boot
```
The initial installation might take a bit time. You can check if your certificate is ready with the following commands:
```
$ kubectl get certificate -A -w
NAMESPACE   NAME          READY   SECRET            AGE
default     certificate   True    certificate-tls   4m18s
$ kubectl describe certificate
...
Events:
  Type    Reason     Age    From          Message
  ----    ------     ----   ----          -------
  Normal  Issuing    4m59s  cert-manager  Issuing certificate as Secret does not exist
  Normal  Generated  4m59s  cert-manager  Stored new private key in temporary Secret resource "certificate-dwtmf"
  Normal  Requested  4m59s  cert-manager  Created new CertificateRequest resource "certificate-q9kmt"
  Normal  Issuing    65s    cert-manager  The certificate has been successfully issued
```
Once your certificate is ready, you can access https://platform-dev.tsuyo.org and https://<subdomain>.platform-dev.tsuyo.org with HTTPS.
You can use a sample application for your test as follows:
```
$ curl -s https://raw.githubusercontent.com/tsuyo/cert-boot/main/sample/hello-k8s-world.yaml | sed 's/<YOURHOST>/hello-world.platform-dev.tsuyo.org/g' | kubectl apply -f -
$ curl https://hello-world.platform-dev.tsuyo.org/
Hello, world!
Version: 1.0.0
Hostname: hello-world-6bfb8dfdb8-6htq2
```

# For Developers
```
$ git clone https://github.com/tsuyo/cert-boot.git
$ cd cert-boot
$ helm dependency update
$ vi values.yaml # you can modify any templates in this repo as well
$ helm upgrade cert-boot -i --create-namespace -n cert-boot -f values.yaml ./chart
$ helm package chart --destination docs
$ helm repo index docs
```
