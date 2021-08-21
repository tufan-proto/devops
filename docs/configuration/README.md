# Configuration

## Architecture

We configure and deploy a [backstage app](https://github.com/acuity-sr/bkstg-one) to an Azure Kubernetes (AKS) Cluster.

The [AKS workshop architecture](https://docs.microsoft.com/en-us/learn/modules/aks-workshop/01-introduction) is adapted to fit our needs

![Architecture](./images/02_aks_workshop.svg)


### Phase 1
For a phase 1 implementation, the architecture has been
adapted in the following ways:

1. [backstage](https://backstage.io) makes it rather difficult to separate the front-end and back-end of the application. Specifically, the code modifications required
add to an ongoing maintenance and upgrade cost that we'd 
rather avoid. So we deploy one k8s service/deployment for both. 
2. We currently use sqlite to keep things simple as we get the rest of the flow going. The eventual goal is to use an Azure Managed postgres implementation
3. The nginx-ingress and lets-encrypt services are not fully functional as of this writing.

### Roadmap
- [ ] nginx-ingress
- [ ] lets-encrypt
- [ ] Azure managed postgres
- [ ] logging
- [ ] ha
- [ ] CDN for front-end assets
- [ ] load-testing

### Future
At some point in the future, we hope to incorporate as much of the security and governance capabilities as appropriate. The [Security and Governance workshop](https://github.com/Azure/sg-aks-workshop) will serve as a starting point for that exercise. The link is only provided here for (future) reference.

## Infrastructure/Configuration inventory

These are the pieces that we'll need to provision/configure as part of our script.
They are also split up by stage of creation, allowing us automate with clear separation
of responsibility.

| Layer      | Azure                    | Kubernetes                  |
| ---------- | ------------------------ | --------------------------- |
| bootstrap  | resource-group           |                             |
|            | service-principal        |                             |
| infra      | azure-networking         |                             |
|            | AKS-cluster              |                             |
|            | ACR                      |                             |
|            | bind AKS-ACR             |                             |
|            | *Azure-postgres          |                             |
|            | *log-analytics workspace |                             |
|            | *AKS monitoring addon    |                             |
| kubernetes |                          | namespace                   |
|            |                          | api-Deployment              |
|            |                          | api-Service                 |
|            |                          | LoadBalancer                |
|            |                          | ui-Deployment               |
|            |                          | ui-Service                  |
|            |                          | ingress                     |
|            |                          | cert-manager                |
|            |                          | ClusterRole(monitoring)     |
|            |                          | api-HorizontalPodAutoscaler |
