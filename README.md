# argocd-helmchart

## pre-reqs

1. Deploy `nginx ingress`
    ```
    helm upgrade --install ingress-nginx ingress-nginx \
        --repo https://kubernetes.github.io/ingress-nginx \
        --namespace ingress-nginx --create-namespace
    ```
    > **Note:**: You will then need to create a CNAME record for `argocd.sampledomain.com` pointing to the nginx ingress ELB.
2. Deploy `cert manager`
    * install cert-manager
    ```
    helm repo add jetstack https://charts.jetstack.io
    helm repo update
    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.crds.yaml
    helm install \
        cert-manager jetstack/cert-manager \
        --namespace cert-manager --create-namespace \
        --version v1.12.0
    ```
    * create certificate
    ```
    kubectl apply -f - <<EOF
    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
        name: sample-ca
        namespace: cert-manager
    spec:
        isCA: true
        duration: 43800h
        commonName: sampledomain.com
        secretName: sample-key-pair
        privateKey:
            algorithm: ECDSA
            size: 256
        issuerRef:
            name: selfsigned
            kind: ClusterIssuer
            group: cert-manager.io
    EOF
    ```
    * create selfsigned cluster issuer
    ```
    kubectl apply -f - <<EOF
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
        name: selfsigned
    spec:
        ca:
            secretName: sample-key-pair
    EOF
    ```

## deploy argocd

```
helm install <release name> . --values values.yaml --namespace argocd --create-namespace
```
> **Note:**: `release name` can be anything, but it needs to be unique.