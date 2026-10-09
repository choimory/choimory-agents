## DevOps 주석 규칙

### 적용대상

- Terraform
- Kubernetes Manifest
- Helm
- Kustomize
- ArgoCD
- GitHub Actions
- Jenkins Pipeline
- Dockerfile
- Shell Script
- YAML 기반 CI/CD 및 GitOps 설정

### Comment

- 모든 주요 리소스 또는 설정 블록에는 해당 리소스의 용도와 역할을 설명하는 주석을 작성한다.
- 환경별로 값이나 동작이 달라지는 설정에는 해당 값이 어떤 환경에서 사용되는지 명시한다.
- 네트워크, 보안, 인증, 스토리지 등 운영에 영향을 주는 주요 설정에는 설정 목적을 간략하게 작성한다.
- 외부 시스템, 다른 리소스 또는 다른 Manifest와 연계되는 설정에는 연관 관계를 설명한다.
- 단순히 설정값을 그대로 읽어주는 주석은 작성하지 않는다.
- 코드나 설정을 흐름 단위로 구분하고, 각 흐름의 시작 부분에 해당 작업의 목적을 설명하는 간단한 한글 주석을 작성한다.
- 특별한 이유로 기본값이나 일반적인 구성과 다르게 설정한 경우 반드시 이유를 주석으로 남긴다.
- 삭제 또는 변경 시 장애 가능성이 있는 설정에는 운영상 주의사항을 명시한다.
- 비밀번호, API Key, Access Key 등의 민감정보는 주석에도 직접 작성하지 않는다.
- 하위 주석 양식은 코드 컨벤션과는 관련이 없으며, 주석 작성 방식만 참고한다.

### Terraform

```hcl
# 운영 Kubernetes 워커 노드가 위치하는 Private Subnet
resource "ncloud_subnet" "prod_k8s_private" {
  vpc_no         = ncloud_vpc.prod.id
  subnet         = "10.10.10.0/24"
  zone           = "KR-1"
  network_acl_no = ncloud_network_acl.prod.id
  subnet_type    = "PRIVATE"
  usage_type     = "GEN"
}

# Private Subnet의 외부 통신을 NAT Gateway로 전달한다.
# Kubernetes 노드에서 Container Registry, Git Repository 등으로
# 아웃바운드 통신하기 위해 필요하다.
resource "ncloud_route" "prod_nat_route" {
  route_table_no         = ncloud_route_table.prod_private.id
  destination_cidr_block = "0.0.0.0/0"
  target_type            = "NATGW"
  target_no              = ncloud_nat_gateway.prod.id
}
```

### Kubernetes Manifest

```yaml
# API Gateway 애플리케이션 배포
# 외부 요청을 받아 내부 Backend API로 전달한다.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: application
spec:
  replicas: 2

  selector:
    matchLabels:
      app: api-gateway

  template:
    metadata:
      labels:
        app: api-gateway

    spec:
      containers:
        - name: api-gateway
          image: registry.example.com/api-gateway:1.0.0

          # 애플리케이션 HTTP 요청 처리 포트
          ports:
            - containerPort: 8080

          env:
            # 실제 Redis 접속정보는 Kubernetes Secret에서 주입한다.
            - name: REDIS_HOST
              valueFrom:
                secretKeyRef:
                  name: redis-secret
                  key: host
```

### Kubernetes Service / Ingress

```yaml
# ALB에서 Kubernetes Node를 대상으로 트래픽을 전달하기 때문에
# Service Type은 NodePort를 사용한다.
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
spec:
  type: NodePort

  selector:
    app: api-gateway

  ports:
    - port: 80
      targetPort: 8080
```

### Kustomize

```yaml
# QA 환경 전용 설정
# 공통 Base Manifest를 기준으로 이미지 태그와 Replica 수만 변경한다.
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

images:
  # Jenkins 빌드 이후 GitOps Repository에서 태그가 자동 변경된다.
  - name: registry.example.com/im-cms
    newTag: "125"

patches:
  # QA 환경은 운영보다 적은 Replica를 사용한다.
  - path: deployment-patch.yaml
```

### ArgoCD

```yaml
# QA 환경 im-cms 애플리케이션 배포 관리
# GitOps Repository의 qa overlay 변경사항을 Kubernetes에 반영한다.
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: im-cms-qa
  namespace: argocd

spec:
  source:
    repoURL: https://git.example.com/gitops/im-cms.git
    targetRevision: main
    path: saas/overlays/qa

  destination:
    server: https://kubernetes.default.svc
    namespace: application

  syncPolicy:
    automated:
      # Git에서 제거된 리소스를 Kubernetes에서도 자동 삭제한다.
      prune: true

      # 클러스터에서 수동 변경된 설정을 Git 상태로 복구한다.
      selfHeal: true
```

### Helm

```yaml
# QA 환경 ArgoCD 설정
server:
  replicas: 1

  service:
    # 외부에 직접 노출하지 않고 내부 Ingress를 통해 접근한다.
    type: ClusterIP

configs:
  params:
    # 사내 환경의 TLS 종료 구조 때문에
    # ArgoCD 내부 서버는 HTTP로 동작한다.
    server.insecure: true
```

### GitHub Actions

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 애플리케이션 소스코드를 Checkout한다.
      - name: Checkout
        uses: actions/checkout@v4

      # Container Image를 빌드한다.
      - name: Build Image
        run: |
          docker build \
            -t registry.example.com/im-cms:${{ github.run_number }} .

      # Container Registry에 빌드된 이미지를 Push한다.
      - name: Push Image
        run: |
          docker push \
            registry.example.com/im-cms:${{ github.run_number }}

      # GitOps Repository의 이미지 태그를 변경한다.
      # ArgoCD가 변경사항을 감지하여 Kubernetes에 자동 반영한다.
      - name: Update GitOps
        run: |
          ./scripts/update-image-tag.sh \
            im-cms \
            ${{ github.run_number }}
```

### Jenkins Pipeline

```groovy
pipeline {
    agent any

    stages {

        // 애플리케이션 이미지를 생성한다.
        stage('Build') {
            steps {
                sh '''
                    nerdctl build \
                      -t ${REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        // 생성된 이미지를 Container Registry에 업로드한다.
        stage('Push') {
            steps {
                sh '''
                    nerdctl push \
                      ${REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        // GitOps Repository의 이미지 태그를 변경한다.
        // 이후 ArgoCD가 변경사항을 감지하여 Kubernetes에 반영한다.
        stage('Update GitOps') {
            steps {
                sh '''
                    cd /home/jenkins/gitops/${SERVICE_NAME}

                    kustomize edit set image \
                      ${REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}

                    git add .
                    git commit -m "deploy: ${SERVICE_NAME} ${BUILD_NUMBER}"
                    git push
                '''
            }
        }
    }
}
```

### Dockerfile

```dockerfile
# 애플리케이션 빌드를 위한 Stage
FROM gradle:8-jdk21 AS builder

WORKDIR /app

# 의존성 캐시를 최대한 활용하기 위해
# 빌드 설정 파일을 먼저 복사한다.
COPY build.gradle settings.gradle ./
COPY src ./src

RUN gradle clean build -x test


# 실제 애플리케이션 실행을 위한 Runtime Stage
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=builder /app/build/libs/app.jar app.jar

# 애플리케이션 HTTP 포트
EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Shell Script

```bash
#!/bin/bash

# 명령 실행 실패 시 즉시 스크립트를 종료한다.
set -e

SERVICE_NAME="$1"
IMAGE_TAG="$2"

# GitOps Repository에서 대상 서비스 디렉터리로 이동한다.
cd "/home/jenkins/gitops/${SERVICE_NAME}"

# 배포할 Container Image Tag를 변경한다.
kustomize edit set image \
  "registry.example.com/${SERVICE_NAME}:${IMAGE_TAG}"

# 변경된 GitOps 설정을 Repository에 반영한다.
git add .
git commit -m "deploy: ${SERVICE_NAME} ${IMAGE_TAG}"
git push
```

### 주석 작성 기준

좋은 주석:

```yaml
# ALB가 Kubernetes Node를 Target으로 사용하기 때문에 NodePort로 노출한다.
type: NodePort
```

```hcl
# Private Subnet의 인터넷 아웃바운드 통신을 위해 NAT Gateway를 사용한다.
destination_cidr_block = "0.0.0.0/0"
```

```yaml
# 운영 환경에서는 장애 대응을 위해 최소 3개의 Replica를 유지한다.
replicas: 3
```

피해야 할 주석:

```yaml
# Replica 개수
replicas: 3
```

```hcl
# CIDR 설정
cidr_block = "10.10.0.0/16"
```

```yaml
# 포트 8080
containerPort: 8080
```

주석은 **설정값이 무엇인지**보다 **왜 해당 설정을 사용했는지, 어떤 역할을 하는지**를 설명하는 것을 기본 원칙으로 한다.