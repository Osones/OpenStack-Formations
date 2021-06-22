# KUBERNETES : Gestion des placements de pods

### _Taints_ et _Tolerations_

Un nœud avec un taint empêche l'exécution sur lui-même des pods qui ne tolèrent pas ce "taint"

- Les _taints_ et _tolerations_ fonctionnent ensemble
- Les _taints_ sont appliqués aux nœuds
- Les _tolerations_ sont décrites aux niveau des pods
- On peut mettre plusieurs _taints_ sur un nœuds (il faudra que le pod tolère tous ces _taints_)
- Les _taints_ sont définis ainsi : `key=value:Effect`


### _Taints_ et _Tolerations_ : Champ "Effect"

Ce champ peut avoir 3 valeurs : `NoSchedule`, `PreferNoSchedule`, `NoExecute`

- NoSchedule : Contrainte forte, seuls les pods supportant le "taint" pourront s'exécuter sur le nœud.
- PreferNoSchedule: Contrainte faible, le scheduler de kubernetes **évitera** de placer un pod qui ne tolère pas ce taint sur le nœud, mais pourra le faire si besoin
- NoExecute : Contrainte forte, les pods seront expulsés du nœud / ne pourront pas s'exécuter sur le nœuds


### _Taints_ et _Tolerations_ : Operateur

- Par valeur par défaut est `Equal` (`key=value:Effect`)
- Mais peut avoir aussi comme valeur `Exist` (`keyExist:Effect`) 


### _Taints_ et _Tolerations_ : Utilisation des _taints_

En ligne de commande

- Ajouter un taint
    - `kubectl taint nodes THENODE special=true:NoExecute`
    - `kubectl taint node node1 node-role.kubernetes.io/master="":NoSchedule`
    - `kubectl taint node node2 test:NoSchedule`

- Supprimer un taint
    - `kubectl taint nodes THENODE special=true:NoExecute-`
    - `kubectl taint node node1 node-role.kubernetes.io/master-`
    - `kubectl taint node node2 test:NoSchedule-`

- Lister les tains d'un nœud
    - `kubectl get nodes THENODE -o jsonpath="{.spec.taints}"`
    - `[{"effect":"NoSchedule","key":"node-role.kubernetes.io/master"}]`


### _Taints_ et _Tolerations_ : Utilisation des _tolerations_

- Les _tolerations_ peuvent être décrite au niveau des pods ou au niveau des templates de pods dans les replicaset, daemonset, statefulset et deployment.

- Ces _tolerations_ permettront aux pods de s'exécuter sur les nœuds qui ont le "taint" en correspondance.


### _Taints_ et _Tolerations_ : Exemples 

- Quand il n'y a pas de values dans le taint

```yaml
apiVersion: v1
kind: Pod
metadata:
...
spec:
  tolerations:
    - key: node-role.kubernetes.io/master
      effect: NoSchedule
```

### _Taints_ et _Tolerations_ : Exemples (suite)

- Quand il y a une values

```yaml
apiVersion: v1
kind: Pod
metadata:
...
spec:
  tolerations:
    - key:  special
      value: "true"
      effect: NoExecute
      Operator: Equal
```


### _Taints_ et _Tolerations_ : Cas particulier

Une clé vide avec un opérateur Exist fera en sorte que le pod s'exécutera sur tous les nœuds quelques soit leurs "taint"

exemple :

```yaml
tolerations:
- operator: "Exists"
```


### nodeSelector

- Champs clé valeur au niveau des podSpecs
- Pour que le pod puisse s'exécuter il faut que le nœud ait l'ensemble des labels correspondants
  
Exemples 

- Pose du label sur un nœud
    - `kubectl label nodes <node-name> <label-key>=<label-value>`
    - ex: `kubectl label nodes node1 disktype=ssd`

- Utilisation dans un pod


```yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disktype: ssd

```


### Affinité / Anti-affinité

Ce système permet de gérer très finement les règles de placement des pods en regard de la simplicité du nodeSelector

- Le langage permettant d'exprimer les affinités / anti-affinités est riche de possibilités
- Possibilité de d'écrire des préférences `soft` ou `hard`  (pod déployé malgré tout)
- Contraintes dépendantes de labels présents dans d'autres pods

### Node Affinity

- Égal conceptuellement au nodeSelector, mais avec la possibilité de faire du **soft** (should) ou **hard** (must)
- Soft : `preferredDuringSchedulingIgnoredDuringExecution`
- Hard :  `requiredDuringSchedulingIgnoredDuringExecution`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In # (NotIn, Exists, DoesNotExist, Gt, Lt)
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In # (NotIn, Exists, DoesNotExist, Gt, Lt)
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```


### Autre exemple

```yaml
...
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd
...
```

### Pod Affinity

```yaml
...
  template:
    metadata:
      labels:
        app: redis
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - redis
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
...

```

### Pod anti-Affinity

```yaml
...
template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - redis
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        
    ...

```
