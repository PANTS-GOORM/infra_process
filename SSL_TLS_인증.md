# SSL/TLS 인증

## Cert-Manager 및 Nginx 컨트롤러 설치

1. 네임스페이스 추가   
   ```
   kubectl create namespace cert-manager  
   kubectl create namespace ingress-nginx  
   ```  

2. 차트 추가

  - Helm 차트 저장소 추가

    ```
    helm repo add jetstack https://charts.jetstack.io
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    ```

3. Cert-Manager 설치
     
   ```
   helm install cert-manager jetstack/cert-manager \
   --namespace cert-manager \
   --version v1.12.0 \
   --set installCRDs=true
   ```

4. Nginx Controller 설치
   
   ``` 
   helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx
   ```

   참고: aws용 nginx
   
   ```
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/aws/deploy.yaml
   ```

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
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  ingressClassName: nginx  # 사용 중인 Ingress 컨트롤러에 맞게 변경
  rules:
  - host: argocd.yourdomain.com  # 실제 도메인으로 변경
    http:
      paths:
      - path: /argocd
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 80
  tls:
  - hosts:
    - argocd.yourdomain.com  # 실제 도메인으로 변경
    secretName: argocd-server-tls  # 필요에 따라 변경 가능
```

  - 이 파일을 ingress.yaml로 저장하고, 다음 명령어를 사용하여 적용   
    ``` kubectl apply -f argocd-ingress.yaml ```
