# Practicas-SA-B-202203856-gitops

Repositorio GitOps de **sa-platform** (Práctica 8 — [ELFBIBv/Practicas-SA-B-202203856](https://github.com/ELFBIBv/Practicas-SA-B-202203856)). Contiene únicamente manifiestos declarativos: el estado deseado del clúster. Nadie aplica cambios aquí a mano con `kubectl`/`helm`; el único componente con permiso de escritura hacia el clúster es **ArgoCD**, que sincroniza este repositorio de forma automática.

## Estructura

```
charts/sa-platform/     Copia del chart de Helm, sincronizada desde /P8/charts/sa-platform
                         del repo de código en cada promoción.
argocd/application.yaml Recurso Application de ArgoCD que apunta a este repo y namespace sa-p5.
```

## Cómo llega un cambio hasta acá

1. El pipeline de CI del repo de código construye y publica las imágenes en GHCR.
2. Si se dispara manualmente con el checkbox `deploy`, el job `promote-to-gitops` sincroniza el chart hacia `charts/sa-platform/`, actualiza el tag de imagen de cada servicio al commit recién publicado, y abre un Pull Request contra `main` de este repositorio.
3. Una persona revisa y mergea ese PR (o se automerge según la política que se configure).
4. ArgoCD, que vigila `main` de este repo, detecta el cambio y sincroniza el clúster. El pipeline de CI **nunca** tiene credenciales de Azure ni acceso directo al clúster.

## Aplicación de ArgoCD

```bash
kubectl apply -n argocd -f argocd/application.yaml
```

Requiere que ArgoCD ya esté instalado en el clúster y que el repositorio (público) esté agregado o accesible sin credenciales adicionales.

## Nota sobre `ghcr-secret`

El chart referencia `global.imagePullSecrets: [ghcr-secret]` para poder descargar las imágenes privadas de GHCR. Ese secreto **no** se versiona aquí en texto plano; se crea una sola vez durante el bootstrap del clúster (ver el procedimiento de Terraform/ArgoCD del repo de código) o se migra a Sealed Secrets/External Secrets como parte de la Práctica 8.
