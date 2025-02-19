### Get your token. from Cloudflare
[CloudFlare Api Token](https://dash.cloudflare.com/profile/api-tokens)
> Create Token > Custom token(Get started) > Permissions: Zone, SSL and Certificates, Edit > <br> Zone Resources: Include, All zones > Continue to summary > Copy your **API-key** and save it to `cfapi-token.key`

### Encoding cfapi-token to base64
```bash
$ base64 ./cfapi-token.key
$(SOMEVALUE)
```

### Set cfapi-token-secret.yaml
```yaml
apiVersion: v1
data:
  # the key must be encoded in base64
  key: $(SOMEVALUE)
kind: Secret
metadata:
  creationTimestamp: null
  name: cfapi-token
  namespace: istio-system
```


### Install CRD & Deploy [CloudFlare:Origin-CA-Issuer](https://github.com/cloudflare/origin-ca-issuer)
```bash
$ git clone https://github.com/cloudflare/origin-ca-issuer.git
$ kubectl apply -f ./origin-ca-issuer/deploy/crds
$ kubectl apply -f ./origin-ca-issuer/deploy/manifests
```

### Install [cert-manager](https://cert-manager.io/docs/installation/)
```bash
$ kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.17.0/cert-manager.yaml
$ kubectl get pods --namespace cert-manager
```

### Set certificate.yaml
```yaml
# [[file:../../README.org::*Creating our first certificate][Creating our first certificate:1]]
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-com
  namespace: istio-system
spec:
  # The secret name where cert-manager should store the signed certificate
  secretName: example-com-tls
  dnsNames:
    - example.com
    - "*.example.com"
  # Duration of the certificate
  duration: 168h
  # Renew a day before the certificate expiration
  renewBefore: 24h
  # Reference the Origin CA Issuer you created above, which must be in the same namespace.
  issuerRef:
    group: cert-manager.k8s.cloudflare.com
    kind: OriginIssuer
    name: prod-issuer
# Creating our first certificate:1 ends here
```

### Deploy certificate and Auto Create secret
```bash
$ kubectl apply -f ./issuer-secret
```

### Check Secret
```bash
$ kubectl get certificate -A
NAMESPACE      NAME         READY   SECRET           AGE
istio-system   example-com   True    example-com-tls   29s
```

#### References
- [origin-ca-issuer](https://github.com/cloudflare/origin-ca-issuer/blob/trunk/deploy/crds/cert-manager.k8s.cloudflare.com_clusteroriginissuers.yaml)
- [Installing cert-manager](https://cert-manager.io/docs/installation/)