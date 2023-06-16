---
layout: default
title: Gitlab Runner Operator
parent: 标准流程
nav_order: 2
---

## 安装方式

### operator

- [原始文档地址](https://operatorhub.io/operator/gitlab-runner-operator)
- install目录本地安装，v0.24.0版本，避免文件拉不下来导致安装失败
- 镜像拉取暂未本地化，可先通过mirrors或者梯子方式解决

## 本地安装步骤说明

1. 安装OLM，openshift默认自带，其他集群需要先安装这个operator包管理器

    ```bash
    kubectl apply -f install/cert-manager
    kubectl apply -f install/crds.yaml
    kubectl apply -f install/olm.yaml

    kubectl rollout status -w deployment/olm-operator --namespace=olm
    kubectl rollout status -w deployment/catalog-operator --namespace=olm
    ```

2. 安装gitlab-runner-operator

    ```bash
    kubectl apply -f install/gitlab-runner-operator.yaml

    ##可通过以下命令验证
    kubectl get csv -n operators
    ```

3. 实例化gitlab-runner

    ```bash
    cat > gitlab-runner-secret.yml << EOF
    apiVersion: v1
    kind: Secret
    metadata:
    name: gitlab-runner-secret
    type: Opaque
    stringData:
    runner-registration-token: REPLACE_ME # your project runner secret
    EOF

    kubectl apply -f gitlab-runner-secret.yml


    cat > gitlab-runner.yml << EOF
    apiVersion: apps.gitlab.com/v1beta2
    kind: Runner
    metadata:
        name: gitlab-runner
    spec:
        gitlabUrl: https://gitlab.example.com
        buildImage: alpine
        token: gitlab-runner-secret
        tags: openshift
    EOF

    kubectl apply -f gitlab-runner.yml
    ```