# 🚀 CI/CD Pipelines - Automatisation des Déploiements

Guide complet des pipelines CI/CD avec GitLab CI, GitHub Actions et Jenkins.

---

## 📚 Introduction

**CI/CD** = Continuous Integration / Continuous Deployment

**Continuous Integration (CI) :**
- Build automatique
- Tests automatiques
- Code quality checks
- Security scanning

**Continuous Deployment (CD) :**
- Déploiement automatique
- Rollback automatique
- Blue/Green deployments
- Canary releases

---

## 🦊 GitLab CI/CD

### .gitlab-ci.yml - Structure de Base

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_REGISTRY: registry.gitlab.com
  IMAGE_NAME: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG

before_script:
  - echo "Pipeline started"

after_script:
  - echo "Pipeline finished"

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $IMAGE_NAME .
    - docker push $IMAGE_NAME
  only:
    - main
    - develop

test:
  stage: test
  image: node:18
  script:
    - npm install
    - npm test
  coverage: '/Coverage: \d+\.\d+%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

deploy_staging:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache curl
    - curl -X POST $WEBHOOK_URL
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy_production:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache curl
    - curl -X POST $WEBHOOK_PROD_URL
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual  # Déploiement manuel en production
```

### Variables et Secrets

```yaml
# Variables prédéfinies GitLab
variables:
  CI_COMMIT_REF_NAME: "main"           # Nom de la branche
  CI_COMMIT_SHA: "abc123..."           # Hash du commit
  CI_PROJECT_NAME: "my-project"        # Nom du projet
  CI_REGISTRY: "registry.gitlab.com"   # Registry GitLab
  CI_REGISTRY_IMAGE: "registry.gitlab.com/user/project"

# Secrets (Settings → CI/CD → Variables)
# $DATABASE_PASSWORD
# $API_KEY
# $SSH_PRIVATE_KEY

# Utilisation
deploy:
  script:
    - echo "Deploying with key $API_KEY"
    - ssh -i $SSH_PRIVATE_KEY user@server
```

### Artifacts et Cache

```yaml
# Artifacts (fichiers conservés entre jobs)
build:
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
      - node_modules/
    expire_in: 1 week

test:
  dependencies:
    - build  # Récupère artifacts de "build"
  script:
    - npm test

# Cache (accélère builds)
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/

job_with_cache:
  cache:
    key: npm-cache
    paths:
      - node_modules/
  script:
    - npm ci
```

### Conditions et Rules

```yaml
# only/except (ancien)
deploy:
  only:
    - main
    - /^release-.*$/  # Regex
  except:
    - tags

# rules (moderne)
deploy:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always
    - if: '$CI_COMMIT_TAG'
      when: never
    - when: manual

# Conditions multiples
deploy:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: never
    - if: '$CI_COMMIT_BRANCH == "main"'
      changes:
        - src/**/*
        - Dockerfile
      when: on_success
```

### Templates et Includes

```yaml
# .gitlab-ci.yml (principal)
include:
  - local: '/.gitlab/ci/build.yml'
  - local: '/.gitlab/ci/test.yml'
  - template: Security/SAST.gitlab-ci.yml

# .gitlab/ci/build.yml
.build_template:
  image: docker:latest
  script:
    - docker build -t $IMAGE .

build_dev:
  extends: .build_template
  variables:
    IMAGE: myapp:dev

build_prod:
  extends: .build_template
  variables:
    IMAGE: myapp:prod
```

### Exemple Complet - Application Node.js

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - security
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

# Build Docker image
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  only:
    - main
    - develop

# Unit tests
test:unit:
  stage: test
  image: node:18-alpine
  before_script:
    - npm ci
  script:
    - npm run test:unit
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Integration tests
test:integration:
  stage: test
  image: docker:latest
  services:
    - docker:dind
    - postgres:13
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
  script:
    - docker-compose -f docker-compose.test.yml up -d
    - docker-compose exec -T app npm run test:integration
    - docker-compose down

# Security scan
security:scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --severity HIGH,CRITICAL $IMAGE_TAG
  allow_failure: true

# Deploy staging
deploy:staging:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | ssh-add -
    - mkdir -p ~/.ssh
    - ssh-keyscan $STAGING_SERVER >> ~/.ssh/known_hosts
  script:
    - ssh user@$STAGING_SERVER "docker pull $IMAGE_TAG"
    - ssh user@$STAGING_SERVER "docker-compose up -d"
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# Deploy production (manual)
deploy:production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client kubectl
    - echo "$KUBE_CONFIG" > ~/.kube/config
  script:
    - kubectl set image deployment/app app=$IMAGE_TAG
    - kubectl rollout status deployment/app
  environment:
    name: production
    url: https://example.com
    on_stop: rollback
  only:
    - main
  when: manual

# Rollback production
rollback:
  stage: deploy
  image: alpine:latest
  script:
    - kubectl rollout undo deployment/app
  environment:
    name: production
    action: stop
  when: manual
  only:
    - main
```

---

## 🐙 GitHub Actions

### .github/workflows Structure

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * 1'  # Chaque lundi à 2h
  workflow_dispatch:  # Déclenchement manuel

env:
  NODE_VERSION: 18
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/
          retention-days: 7

  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Download artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist

      - name: Run tests
        run: npm test

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage.xml

  docker:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    steps:
      - name: Deploy to Kubernetes
        env:
          KUBE_CONFIG: ${{ secrets.KUBE_CONFIG }}
        run: |
          echo "$KUBE_CONFIG" > ~/.kube/config
          kubectl set image deployment/app app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:sha-${{ github.sha }}
          kubectl rollout status deployment/app
```

### Secrets et Variables

```yaml
# Secrets (Settings → Secrets and variables → Actions)
# ${{ secrets.AWS_ACCESS_KEY_ID }}
# ${{ secrets.DOCKER_PASSWORD }}
# ${{ secrets.KUBE_CONFIG }}

# Variables d'environnement
# ${{ env.NODE_VERSION }}

# GitHub context
# ${{ github.repository }}      # owner/repo
# ${{ github.ref }}              # refs/heads/main
# ${{ github.sha }}              # Commit SHA
# ${{ github.actor }}            # Username qui a déclenché
# ${{ github.event_name }}       # push, pull_request, etc.
```

### Matrix Strategy

```yaml
# Tester sur plusieurs versions
test:
  runs-on: ${{ matrix.os }}
  strategy:
    matrix:
      os: [ubuntu-latest, windows-latest, macos-latest]
      node: [16, 18, 20]
      exclude:
        - os: macos-latest
          node: 16
    fail-fast: false
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node }}
    - run: npm test
```

### Reusable Workflows

```yaml
# .github/workflows/deploy.yml (réutilisable)
name: Deploy

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      deploy_token:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - name: Deploy
        run: echo "Deploying to ${{ inputs.environment }}"

# .github/workflows/main.yml (appelant)
name: Main

on: push

jobs:
  deploy-staging:
    uses: ./.github/workflows/deploy.yml
    with:
      environment: staging
    secrets:
      deploy_token: ${{ secrets.STAGING_TOKEN }}

  deploy-prod:
    uses: ./.github/workflows/deploy.yml
    with:
      environment: production
    secrets:
      deploy_token: ${{ secrets.PROD_TOKEN }}
```

### Actions Marketplace Populaires

```yaml
# Checkout code
- uses: actions/checkout@v4

# Setup languages
- uses: actions/setup-node@v4
- uses: actions/setup-python@v4
- uses: actions/setup-java@v4
- uses: actions/setup-go@v4

# Docker
- uses: docker/setup-buildx-action@v3
- uses: docker/login-action@v3
- uses: docker/build-push-action@v5

# Cloud
- uses: aws-actions/configure-aws-credentials@v4
- uses: azure/login@v1
- uses: google-github-actions/auth@v1

# Security
- uses: aquasecurity/trivy-action@master
- uses: snyk/actions/node@master

# Notifications
- uses: slackapi/slack-github-action@v1
- uses: 8398a7/action-slack@v3

# Artifacts
- uses: actions/upload-artifact@v3
- uses: actions/download-artifact@v3

# Cache
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

---

## 🔧 Jenkins

### Jenkinsfile - Declarative Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        IMAGE_NAME = "myapp"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        timeout(time: 1, unit: 'HOURS')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                    post {
                        always {
                            junit 'test-results/*.xml'
                            publishHTML([
                                reportDir: 'coverage',
                                reportFiles: 'index.html',
                                reportName: 'Coverage Report'
                            ])
                        }
                    }
                }
                
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                
                stage('Security Scan') {
                    steps {
                        sh 'npm audit'
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh """
                    ssh user@staging-server '
                        docker pull ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} &&
                        docker-compose up -d
                    '
                """
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                message "Deploy to production?"
                ok "Deploy"
            }
            steps {
                sh """
                    kubectl set image deployment/app app=${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    kubectl rollout status deployment/app
                """
            }
        }
    }
    
    post {
        success {
            slackSend(
                color: 'good',
                message: "Build #${env.BUILD_NUMBER} succeeded: ${env.JOB_NAME}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build #${env.BUILD_NUMBER} failed: ${env.JOB_NAME}"
            )
        }
        always {
            cleanWs()
        }
    }
}
```

### Scripted Pipeline

```groovy
// Jenkinsfile (scripted)
node {
    def app
    
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build') {
        sh 'npm ci'
        sh 'npm run build'
    }
    
    stage('Test') {
        try {
            sh 'npm test'
        } catch (err) {
            currentBuild.result = 'UNSTABLE'
        }
    }
    
    stage('Docker Build') {
        app = docker.build("myapp:${env.BUILD_NUMBER}")
    }
    
    stage('Docker Push') {
        docker.withRegistry('https://registry.example.com', 'docker-creds') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
    
    if (env.BRANCH_NAME == 'main') {
        stage('Deploy') {
            input 'Deploy to production?'
            sh 'kubectl apply -f k8s/'
        }
    }
}
```

### Shared Libraries

```groovy
// vars/buildDockerImage.groovy (shared library)
def call(String imageName, String tag = 'latest') {
    sh "docker build -t ${imageName}:${tag} ."
}

// Jenkinsfile (utilisation)
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                buildDockerImage('myapp', env.BUILD_NUMBER)
            }
        }
    }
}
```

### Jenkins Configuration as Code (JCasC)

```yaml
# jenkins.yaml
jenkins:
  systemMessage: "Jenkins configured automatically"
  numExecutors: 2
  
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: "admin"
          password: "${ADMIN_PASSWORD}"
  
  authorizationStrategy:
    globalMatrix:
      permissions:
        - "Overall/Administer:admin"
        - "Overall/Read:authenticated"

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              scope: GLOBAL
              id: "docker-credentials"
              username: "dockeruser"
              password: "${DOCKER_PASSWORD}"
          - string:
              scope: GLOBAL
              id: "slack-token"
              secret: "${SLACK_TOKEN}"

jobs:
  - script: >
      pipelineJob('my-app-pipeline') {
        definition {
          cpsScm {
            scm {
              git {
                remote {
                  url('https://github.com/user/repo.git')
                }
                branch('*/main')
              }
            }
            scriptPath('Jenkinsfile')
          }
        }
      }
```

---

## 📊 Bonnes Pratiques CI/CD

### 1. Pipeline as Code

```yaml
# ✅ Bon : Pipeline versionné avec le code
# .gitlab-ci.yml, .github/workflows/, Jenkinsfile

# ❌ Mauvais : Configuration UI seulement
```

### 2. Fail Fast

```yaml
# ✅ Bon : Tests rapides en premier
stages:
  - lint      # 30 secondes
  - test      # 2 minutes
  - build     # 5 minutes
  - deploy    # 10 minutes

# ❌ Mauvais : Build avant tests
stages:
  - build     # 5 minutes (échoue après si tests KO)
  - test
```

### 3. Cache et Artifacts

```yaml
# ✅ Bon : Réutiliser node_modules
cache:
  paths:
    - node_modules/

# ✅ Bon : Partager build entre jobs
artifacts:
  paths:
    - dist/
```

### 4. Security Scanning

```yaml
# Scan de sécurité
security:
  script:
    - npm audit
    - trivy image myapp:latest
    - snyk test
```

### 5. Environnements Séparés

```yaml
deploy_staging:
  environment: staging
  only: [develop]

deploy_production:
  environment: production
  only: [main]
  when: manual  # Approval requis
```

### 6. Notifications

```yaml
# Slack notification
after_script:
  - 'curl -X POST -H "Content-Type: application/json" 
    -d "{\"text\":\"Pipeline $CI_PIPELINE_STATUS\"}" 
    $SLACK_WEBHOOK_URL'
```

### 7. Rollback Strategy

```yaml
# Kubernetes rollback
rollback:
  script:
    - kubectl rollout undo deployment/app
  when: manual
```

---

## 🎯 Exemples de Pipelines Complets

### Pipeline Python/Flask

```yaml
# .gitlab-ci.yml
image: python:3.11

stages:
  - test
  - build
  - deploy

before_script:
  - python -V
  - pip install -r requirements.txt

test:
  stage: test
  script:
    - pytest --cov=app tests/
    - flake8 app/
    - black --check app/
  coverage: '/(?i)total.*? (100(?:\.0+)?\%|[1-9]?\d(?:\.\d+)?\%)$/'

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

deploy_k8s:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/flask-app app=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
    - kubectl rollout status deployment/flask-app
  only:
    - main
```

### Pipeline Java/Maven

```yaml
# .github/workflows/maven.yml
name: Java CI/CD

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      
      - name: Build with Maven
        run: mvn -B package --file pom.xml
      
      - name: Run tests
        run: mvn test
      
      - name: SonarQube Scan
        run: mvn sonar:sonar -Dsonar.host.url=${{ secrets.SONAR_URL }}
      
      - name: Build Docker
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker push myapp:${{ github.sha }}
```

---

**🎓 Prochaine étape : [Terraform IaC](./terraform.md) →**
