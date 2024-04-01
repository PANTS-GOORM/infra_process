# SSL/TLS 인증

## Cert-Manager 설치

1. Cert-Manager CustomResourceDefinitions (CRDs) 설치   
   ``` kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.0/cert-manager.crds.yaml ```

2. Cert-Manager 네임스페이스 추가   
   ``` kubectl create namespace cert-manager ```  

3. Cert-Manager Helm 차트 추가 및 설치   

  - Helm 차트 저장소 추가    
``` sudo snap install helm --classic ```   
``` helm repo add jetstack https://charts.jetstack.io ```   
``` helm repo update ```

4. Cert-Manager 설치   
   ``` helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.8.0 ```   

## Let's Encrypt를 위한 Issuer 또는 ClusterIssuer 구성

  - Issuer: 특정 네임스페이스에서만 유효합니다.   
  - ClusterIssuer: 클러스터 전체에서 유효합니다.   
    - ClusterIssuer 예시:
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

  - 이 파일을 clusterissuer.yaml으로 저장하고, 다음 명령어를 사용하여 적용   
    ``` kubectl apply -f clusterissuer.yaml ```   

## 인증서 발급을 위한 Ingress 리소스 구성
   - 애플리케이션에 대한 Ingress 리소스를 구성하고, Cert-Manager에 의해 자동으로 인증서를 발급받도록 설정
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: your-application
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  rules:
  - host: yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: your-service-name
            port:
              number: 80
  tls:
  - hosts:
    - yourdomain.com
    secretName: yourdomain-com-tls
```

  - 이 파일을 ingress.yaml로 저장하고, 다음 명령어를 사용하여 적용   
    ``` kubectl apply -f ingress.yaml ```
