# Argo CD 설치

1. 명령줄 도구 설치 (선택사항)   

``` sudo curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64 ```
``` sudo chmod +x /usr/local/bin/argocd ```   

2. Argo CD 서버 설치   

``` kubectl create namespace argocd ```
``` kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml ```

3. Argo CD 웹 UI에 접근   

``` kubectl port-forward svc/argocd-server -n argocd {port}:443 ```   
   - port는 외부 포트

4. 초기 로그인   

  - ID: admin
  - password: ``` kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2 ```

5. Argo CD Server 서비스를 NodePort로 변경하기   

``` kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}' ```

6. NodePort 확인하기   

``` kubectl get svc argocd-server -n argocd ```   
   - 서비스가 성공적으로 NodePort로 변경되었다면, 할당된 NodePort를 확인해야 합니다.  
   - PORT(S) 열을 확인하면 80:XXXXX/TCP,443:YYYYY/TCP와 같은 포맷 확인가능. 여기서 XXXXX 또는 YYYYY가 외부에서 접근할 수 있는 NodePort 번호
