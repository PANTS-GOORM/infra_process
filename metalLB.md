### 1. MetalLB 설치

   - MetalLB를 설치하기 위해 필요한 리소스(네임스페이스, Deployment, ConfigMap 등)를 쿠버네티스 클러스터에 배포

  ```
  kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.6/manifests/namespace.yaml
  kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.6/manifests/metallb.yaml
  ```

### 2. MetalLB에 IP 주소 범위 할당
   
   - MetalLB가 외부에서 접근 가능한 IP 주소를 할당할 수 있도록 IP 주소 범위를 설정해야 합니다.   
   - 이 설정은 ConfigMap을 통해 이루어집니다.   
   - 사용 가능한 IP 주소 범위는 네트워크 환경에 따라 다르므로, 적절한 범위를 선택해야 합니다.   
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    namespace: metallb-system
    name: config
  data:
    config: |
      address-pools:
      - name: default
        protocol: layer2
        addresses:
        - 192.168.1.240-192.168.1.250 # 실제 사용 가능한 ip범위
  ```

   - ConfigMap을 생성하기 위해 위의 내용을 파일로 저장(예: metallb-config.yaml) 후 다음 명령어를 실행

  ```
  kubectl apply -f metallb-config.yaml
  ```

### 3. LoadBalancer 서비스 생성 및 확인
   - MetalLB 설치 및 구성이 완료되면, LoadBalancer 타입의 쿠버네티스 서비스를 생성하여 외부 IP를 할당 가능
   - 이미 LoadBalancer 타입의 서비스가 있다면, MetalLB가 자동으로 IP 주소를 할당

   - 서비스에 할당된 외부 IP 주소를 확인하려면 다음 명령어를 실행

  ```
  kubectl get svc -n <네임스페이스>
  ```
