## Helm 설치하기

Linux에서 Helm 설치하기

최신 버전의 Helm을 설치하기 위한 스크립트를 실행합니다.

``` 
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

helm version 명령어를 실행하여 설치를 확인

``` helm version ```

## Argo CD 설치하기

1. Argo CD를 위한 네임스페이스 생성하기

   ``` kubectl create namespace argocd ```

2. Helm 차트 저장소 추가하기

   ``` helm repo add argo https://argoproj.github.io/argo-helm ```
   
3. Helm을 통해 Argo CD 설치하기

   ``` helm install argocd argo/argo-cd --namespace argocd --version <version> ```

   3-1. 포트 포워딩 설정(옵션)

   ``` kubectl port-forward svc/argocd-server -n argocd 8080:443 ```

4. 초기 로그인   

  - ID: admin   
  - password: ``` kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d ```
    
5. 초기 비밀번호 확인
 
   ``` kubectl exec -it -n argocd deployment/argocd-server -- /bin/bash ```

6. 초기 비밀번호 변경
   
   ```
   argocd login localhost:8080
   WARNING: server certificate had error: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
   Username: admin
   Password:
   'admin:login' logged in successfully
   Context 'localhost:8080' updated
   ```
   ```
   argocd account update-password
   *** Enter password of currently logged in user (admin):  ## 초기 비밀번호
   *** Enter new password for user admin:  	 ## 변경 비밀번호
   *** Confirm new password for user admin:     ## 변경 비밀번호
   Password updated
   Context 'localhost:8080' updated
   ```

7. Argo CD Server 서비스를 NodePort로 변경하기
   
   ``` kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}' ```

8. NodePort 확인하기
   
   ``` kubectl get svc argocd-server -n argocd ```   
   - 서비스가 성공적으로 NodePort로 변경되었다면, 할당된 NodePort를 확인해야 합니다.  
   - PORT(S) 열을 확인하면 80:XXXXX/TCP,443:YYYYY/TCP와 같은 포맷 확인가능. 여기서 XXXXX 또는 YYYYY가 외부에서 접근할 수 있는 NodePort 번호
