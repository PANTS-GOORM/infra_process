# 프로메테우스

## 1. 프로메테우스 배포

  ``` kubectl create namespace monitoring ```

  - monitoring 네임스페이스 생성
    
  ``` helm repo add prometheus-community https://prometheus-community.github.io/helm-charts ```

  - prometheus-community 차트 리포지토리를 추가
    
  ```
  helm upgrade -i prometheus prometheus-community/prometheus \
  --namespace monitoring \
  --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2"
  ```

## 2. CSI 드라이버 IAM역할 생성

  - IAM OIDC 공급자를 클러스터와 연결하기

  ``` eksctl utils associate-iam-oidc-provider --region=ap-northeast-2 --cluster={cluster-name} --approve ```
  
  - AWS EBS를 PV로 사용하려면 접근권한을 설정
    
  ```
  eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster {cluster-name} \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole
  ```

  - IAM역할생성

  ```
  eksctl create addon --name aws-ebs-csi-driver --cluster {cluster-name} --service-account-role-arn arn:aws:iam::{number}:role/AmazonEKS_EBS_CSI_DriverRole --force
  ```

  - ebs csi 확인
    
  ``` eksctl get addon --name aws-ebs-csi-driver --cluster my-cluster ```
  
# 그라파나

  - [공식 홈페이지 메니페스트](https://grafana.com/docs/grafana/latest/setup-grafana/installation/kubernetes/)

``` yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
  name: grafana
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      securityContext:
        fsGroup: 472
        supplementalGroups:
          - 0
      containers:
        - name: grafana
          image: grafana/grafana:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
              name: http-grafana
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /robots.txt
              port: 3000
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 30
            successThreshold: 1
            timeoutSeconds: 2
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 3000
            timeoutSeconds: 1
          resources:
            requests:
              cpu: 250m
              memory: 750Mi
          volumeMounts:
            - mountPath: /var/lib/grafana
              name: grafana-pv
      volumes:
        - name: grafana-pv
          persistentVolumeClaim:
            claimName: grafana-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 3000
  selector:
    app: grafana
  sessionAffinity: None
  type: ClusterIP
```

# 잉그레스
