# Digital Step Flow - Frontend (Kubernetes)

Repositório dedicado aos manifests Kubernetes do **frontend** da solução Digital Step Flow. Contém apenas a definição dos recursos para deploy do frontend em clusters Kubernetes (Kustomize base + overlay dev).

## Objetivo

Este repositório é atualizado automaticamente pelo pipeline de CI/CD do repositório [pucrs-tcc-digital-step-flow-frontend](https://github.com/raphaelmoraes/pucrs-tcc-digital-step-flow-frontend): após build e push da nova imagem, o job **Deploy to Dev** (no `ci.yml`) atualiza a tag da imagem em `k8s/dev/kustomization.yaml` e faz push para este repositório. O Argo CD (ou aplicação de deploy) sincroniza o cluster com as alterações.

## Estrutura

```
k8s/
├── base/          # Recursos base (Namespace, ConfigMap, Nginx ConfigMap, Deployment, Service, PDB, ServiceMonitor)
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── nginx-configmap.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── poddisruptionbudget.yaml
│   ├── servicemonitor.yaml
│   └── kustomization.yaml
└── dev/           # Overlay desenvolvimento (namespace development, tag da imagem dev)
    └── kustomization.yaml
argocd/
└── application.yaml   # Argo CD Application (opcional; pode apontar para este repo ou para o repo do frontend)
```

## Imagem do frontend

O Deployment utiliza a imagem **raphaelmoraes/digital-step-flow-frontend** (construída a partir da imagem base hardened; ver documentação no repositório do frontend). A tag é definida no overlay:

- **dev:** tag de desenvolvimento (ex.: `24.13.0-r1-dev`), atualizada pelo job Deploy to Dev no `ci.yml` do frontend.

## Deploy

### Manual (kubectl + Kustomize)

```bash
# Desenvolvimento
kubectl apply -k k8s/dev
```

### Via Argo CD

Se o Argo CD estiver configurado para usar **este repositório** como source:

- Aplique a Application: `kubectl apply -f argocd/application.yaml`
- Ajuste no `argocd/application.yaml` o `source.repoURL` para a URL deste repositório e o `source.path` para `k8s/dev` (ou `k8s/prod` quando existir overlay de produção).

Se o Argo CD apontar para o repositório do **frontend** (`pucrs-tcc-digital-step-flow-frontend`) e o path `k8s`, os manifests serão os do frontend; este repositório (frontend-k8s) pode ser usado como repositório separado apenas para os overlays de ambiente, conforme a estratégia adotada.

## Dependências

- **Kubernetes** (cluster com Kustomize suportado, ex.: 1.21+).
- **Imagem do frontend** publicada no registro (Docker Hub ou outro) e acessível pelo cluster.
- Para ServiceMonitor: **Prometheus Operator** (ou compatível) no cluster, se for usar monitoramento.

## Relação com outros repositórios

- **pucrs-tcc-digital-step-flow-frontend:** código da aplicação, Dockerfile, CI/CD. O pipeline desse repositório atualiza a tag da imagem neste repositório (frontend-k8s) no arquivo `k8s/dev/kustomization.yaml`.
