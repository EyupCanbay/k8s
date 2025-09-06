# Authentication
**Authentication** konusuyla ilgili dosyalara buradan erişebilirsiniz.



**Key ve CSR oluşturma**
```
$ openssl genrsa -out ozgurozturk.key 2048 

$ openssl req -new -key ozgurozturk.key -out ozgurozturk.csr -subj "/CN=ozgur@ozgurozturk.net/O=DevTeam"
```

**CertificateSigningRequest oluşturma**

```
$ cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: ozgurozturk
spec:
  groups:
  - system:authenticated
  request: $(cat ozgurozturk.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```


**PoweShell ile CertificateSigningRequest oluşturma**

```
--CSR dosyasını tam yolunu belirterek oku ve Base64'e çevir
$base64_request = [Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\Users\canbay\desktop\k8s\authetication\minikubecrt\eyupcanbay.csr"))

--YAML'ı oluştur ve kubectl ile Kubernetes'e gönder
$csr_yaml = @"
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: eyupcanbay
spec:
  groups:
  - system:authenticated
  request: $base64_request
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
"@

$csr_yaml | kubectl apply -f -
```

**CSR onaylama ve crt'yi alma**

```
$ kubectl get csr

$ kubectl certificate approve ozgurozturk

$ kubectl get csr ozgurozturk -o jsonpath='{.status.certificate}' | base64 -d >> ozgurozturk.crt 
```

**kubectl config ayarları**

```
$ kubectl config set-credentials ozgur@ozgurozturk.net --client-certificate=ozgurozturk.crt --client-key=ozgurozturk.key

$ kubectl config set-context ozgurozturk-context --cluster=minikube --user=ozgur@ozgurozturk.net

$ kubectl config use-context ozgurozturk-context
```