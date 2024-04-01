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
    email: your-email@example.com  # 실제 이메일 주소로 변경
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx  # 사용 중인 Ingress 컨트롤러에 맞게 변경할 수 있음
```

  - 이 파일을 clusterissuer.yaml으로 저장하고, 다음 명령어를 사용하여 적용   
    ``` kubectl apply -f clusterissuer.yaml ```   

## 인증서 발급을 위한 Ingress 리소스 구성
   - 애플리케이션에 대한 Ingress 리소스를 구성하고, Cert-Manager에 의해 자동으로 인증서를 발급받도록 설정   
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
spec:
  ingressClassName: nginx  # 사용 중인 Ingress 컨트롤러에 맞게 변경
  rules:
  - host: yourdomain.com  # 실제 도메인으로 변경
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 80
  tls:
  - hosts:
    - yourdomain.com  # 실제 도메인으로 변경
    secretName: argocd-server-tls  # 필요에 따라 변경 가능
```

  - 이 파일을 ingress.yaml로 저장하고, 다음 명령어를 사용하여 적용   
    ``` kubectl apply -f ingress.yaml ```

## Certificate 리소스 확인 및 생성   
   - Certificate 리소스가 올바르게 생성되었는지 확인하고, 없다면 생성   
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: argocd-server-tls
  namespace: argocd
spec:
  secretName: argocd-server-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - wordsketch.site
```
   - YAML 파일을 argocd-cert.yaml과 같은 이름으로 저장한 후, 다음 명령어로 적용   
     ``` kubectl apply -f argocd-cert.yaml ```
