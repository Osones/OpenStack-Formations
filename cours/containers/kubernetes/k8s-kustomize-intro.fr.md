
# KUBERNETES : Introduction à Kustomize
 
### kubernetes : kustomize

- Kustomize permet de transformer des manifestes yaml contenant des ressources kubernetes
- Les manifestes d'origine restent inchangés et donc utilisables tels quels (`kubectl apply/create/delete -f`)
- On créé des `kustomizations`
- Une `kustomizations` peut se voir comme une superposition (overlay) (ajout ou modification)
- Les `kustomizations` sont définies dans un fichier `kustomizations.yaml`

### kubernetes : kustomizations

- Une kustomization peut contenir:
   - d'autres kustomizations
   - des ressources kubernetes définies en yaml
   - des patchs de ressources kubernetes
   - des ajouts de `labels`ou `annotations` pour toutes les ressources
   - des définitions de `configmaps` ou `secrets`

### kubernetes : kustomizations : exemple 1

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
patchesStrategicMerge:
- scale-deployment.yaml
- ingress-hostname.yaml
resources:
- deployment.yaml
- service.yaml
- ingress.yaml
```

### kubernetes : kustomizations : exemple 2

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
commonLabels:
  add-this-to-all-my-resources: please
patchesStrategicMerge:
- prod-scaling.yaml
- prod-healthchecks.yaml
bases:
- api/
- frontend/
- db/
- github.com/example/app?ref=tag-or-branch
resources:
- ingress.yaml
- permissions.yaml
configMapGenerator:
- name: appconfig
  files:
  - global.conf
  - local.conf=prod.conf
```

### kubernetes : kustomizations : Glossaire

<https://kubectl.docs.kubernetes.io/references/kustomize/glossary/>

- `base` : Une `base` est une kustomisation référencée par une autre kustomisation
- `overlay` : un `overlay`  est une kustomization qui dépend d'une autre kustomization
- `kustomization` : Une `kustomization` peut être à la fois une `base` et un `overlay`
- `patch` : un `patch` décrit comment modifier une ressource existante
- `variant` : une `variante` est le résultat, dans un cluster, de l'application d'un `overlay` à une `base`


### kubernetes : kustomizations : Structure

```bash
├── base
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
└── overlays
    ├── dev
    │   ├── kustomization.yaml
    │   └── patch.yaml
    ├── prod
    │   ├── kustomization.yaml
    │   └── patch.yaml
    └── staging
        ├── kustomization.yaml
        └── patch.yaml
```

