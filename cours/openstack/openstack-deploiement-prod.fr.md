# OpenStack en production et opérations

## Bonnes pratiques pour un déploiement en production

### Quels composants dois-je installer ?

-   Keystone est indispensable
-   L’utilisation de Nova va de paire avec Glance et Neutron
-   Cinder et Swift s'apprécient en fonction des besoins de stockage
-   Swift peut être utilisé indépendemment des autres composants
-   Heat coûte peu
-   Les services plus haut niveau s'évaluent au cas par cas

<https://docs.openstack.org/arch-design/>

### Penser dès le début aux choix structurants

-   Distribution et méthode de déploiement
-   Politique de mise à jour
-   Drivers/backends : hyperviseur, stockage block, etc.
-   Réseau : quelle architecture et quels drivers

### Les différentes méthodes d’installation

-   DevStack est à oublier pour la production
-   Le déploiement à la main comme vu précédemment n’est pas recommandé car peu maintenable
-   Distributions OpenStack packagées et prêtes à l’emploi
-   Distributions classiques et gestion de configuration
-   Déploiement continu

### Mises à jour entre versions majeures

-   OpenStack supporte les mises à jour N $\rightarrow$ N+1
-   Swift : très bonne gestion en mode *rolling upgrade*
-   Autres composants : tester préalablement avec vos données
-   Lire les release notes
-   Cf. articles de blog du CERN <https://techblog.web.cern.ch/techblog/>

### Mises à jour dans une version stable

-   Fourniture de correctifs de bugs majeurs et de sécurité
-   OpenStack intègre ces correctifs sous forme de patchs dans la branche stable
-   Publication de *point releases* intégrant ces correctifs issus de la branche stable
-   Durée variable du support des versions stables, dépendant de l’intérêt des intégrateurs

### Assigner des rôles aux machines

Beaucoup de documentations font référence à ces rôles :

-   Controller node : APIs, BDD, AMQP
-   Network node : DHCP, routeur, IPs flottantes
-   Compute node : Hyperviseur/pilotage des instances

Ce modèle simplifié n’est pas HA.

### Haute disponibilité

Haute disponibilité du IaaS

-   MySQL/MariaDB, RabbitMQ : HA classique (Galera, Clustering)
-   Les services APIs sont stateless et HTTP : scale out et load balancers
-   La plupart des autres services OpenStack sont capables de scale out également

Guide HA : <https://docs.openstack.org/ha-guide/>

Conférences par Florian Haas, Hastexo : <https://www.openstack.org/community/speakers/profile/398/florian-haas>

### Haute disponibilité de l’agent L3 de Neutron

-   *Distributed Virtual Router* (DVR)
-   L3 agent HA (VRRP)

### Considérations APIs

-   Des URLs uniformes pour toutes les APIs :
    - Utiliser un reverse proxy
    - Mettre à jour le catalogue de services
-   Apache/mod\_wsgi pour servir les APIs lorsque cela est possible (Keystone, etc.)

Guide Operations : <https://docs.openstack.org/openstack-ops/content/>

### Découpage réseau

-   Management network : réseau d’administration
-   Data/instances network : réseau pour la communication inter instances
-   External network : réseau externe, dans l’infrastructure réseau existante
-   Storage network : réseau pour le stockage Cinder/Swift
-   API network : réseau contenant les endpoints API

### Considérations liées à la sécurité

-   Indispensable : HTTPS sur l’accès des APIs à l’extérieur
-   Sécurisation des communications MySQL/MariaDB et RabbitMQ
-   Un accès MySQL/MariaDB par base et par service
-   Un utilisateur Keystone par service
-   Limiter l’accès en lecture des fichiers de configuration (mots de passe, token)
-   Veille sur les failles de sécurité : OSSA (*OpenStack Security Advisory*), OSSN (*... Notes*)

Guide sécurité : <https://docs.openstack.org/security-guide/>

### Segmenter son cloud

-   Host aggregates : machines physiques avec des caractéristiques similaires
-   Availability zones : machines dépendantes d’une même source électrique, d’un même switch, d’un même DC, etc.
-   Regions : chaque région a son API
-   Cells : permet de regrouper plusieurs cloud différents sous une même API

<https://docs.openstack.org/openstack-ops/content/scaling.html#segregate_cloud>

### Host aggregates / agrégats d’hôtes

-   Spécifique Nova
-   L’administrateur définit des agrégats d’hôtes via l’API
-   L’administrateur associe flavors et agrégats via des couples clé/valeur communs
-   1 agrégat $\equiv$ 1 point commun, ex : GPU
-   L’utilisateur choisit un agrégat à travers son choix de flavor à la création d’instance

### Availability zones / zones de disponibilité

-   Spécifique Nova et Cinder
-   Groupes d’hôtes
-   Découpage en termes de disponibilité : Rack, Datacenter, etc.
-   L’utilisateur choisit une zone de disponibilité à la création d’instance
-   L’utilisateur peut demander à ce que des instances soient démarrées dans une même zone, ou au contraire dans des zones différentes

### Régions

-   Générique OpenStack
-   Équivalent des régions d’AWS
-   Un service peut avoir différents endpoints dans différentes régions
-   Chaque région est autonome
-   Cas d’usage : cloud de grande ampleur (comme certains clouds publics)

### Cells / Cellules

-   Spécifique Nova
-   Un seul nova-api devant plusieurs cellules
-   Chaque cellule a sa propre BDD et file de messages
-   Ajoute un niveau de scheduling (choix de la cellule)

### Packaging d’OpenStack - Ubuntu

-   Le packaging est fait dans de multiples distributions, RPM, DEB et autres
-   Ubuntu est historiquement la plateforme de référence pour le développement d’OpenStack
-   Le packaging dans Ubuntu suit de près le développement d’OpenStack, et des tests automatisés sont réalisés
-   Canonical fournit la Ubuntu Cloud Archive, qui met à disposition la dernière version d’OpenStack pour la dernière Ubuntu LTS

### Ubuntu Cloud Archive (UCA)

![Support d'OpenStack dans Ubuntu via l'UCA](images/ubuntu-cloud-archive.png)

### Packaging d’OpenStack dans les autres distributions

-   OpenStack est intégré dans les dépôts officiels de Debian
-   Red Hat propose RHOS/RDO (déploiement basé sur TripleO)
-   Comme Ubuntu, le cycle de release de Fedora est synchronisé avec celui d’OpenStack

### Les distributions OpenStack

-   StackOps : historique
-   Mirantis : Fuel
-   HP Helion : Ansible custom
-   etc.

### TripleO

-   OpenStack On OpenStack
-   Objectif : pouvoir déployer un cloud OpenStack (*overcloud*) à partir d’un cloud OpenStack (*undercloud*)
-   Autoscaling du cloud lui-même : déploiement de nouveaux nœuds compute lorsque cela est nécessaire
-   Fonctionne conjointement avec Ironic pour le déploiement bare-metal

### Déploiement bare-metal

-   Le déploiement des hôtes physiques OpenStack peut se faire à l’aide d’outils dédiés
-   MaaS (Metal as a Service), par Ubuntu/Canonical : se couple avec Juju
-   Crowbar / OpenCrowbar (initialement Dell) : utilise Chef
-   eDeploy (eNovance) : déploiement par des images
-   Ironic via TripleO

### Gestion de configuration

-   Puppet, Chef, CFEngine, Saltstack, Ansible, etc.
-   Ces outils peuvent aider à déployer le cloud OpenStack
-   ... mais aussi à gérer les instances (section suivante)

### Modules Puppet, Playbooks Ansible

-   *Puppet OpenStack* et *OpenStack Ansible* : modules Puppet et playbooks Ansible
-   Développés au sein du projet OpenStack
-   <https://wiki.openstack.org/wiki/Puppet>
-   <https://docs.openstack.org/developer/openstack-ansible/install-guide/>

### Déploiement continu

-   OpenStack maintient un master (trunk) toujours stable
-   Possibilité de déployer au jour le jour le `master` (CD : *Continous Delivery*)
-   Nécessite la mise en place d’une infrastructure importante
-   Facilite les mises à jour entre versions majeures

### Test et validation : Tempest

-   Suite de tests d’un cloud OpenStack
-   Effectue des appels à l’API et vérifie le résultat
-   Est très utilisé par les développeurs via l’intégration continue
-   Le déployeur peut utiliser Tempest pour vérifier la bonne conformité de son cloud
-   Cf. aussi Rally

## Gérer les problèmes

### Les réflexes en cas d’erreur ou de comportement erroné

-   Travaille-t-on sur le bon projet ?
-   Est-ce que l’API renvoie une erreur ? (le dashboard peut cacher certaines informations)
-   Si nécessaire d’aller plus loin :
    -   Regarder les logs sur le cloud controller (/var/log/\<composant\>/\*.log)
    -   Regarder les logs sur le compute node et le network node si le problème est spécifique réseau/instance
    -   Éventuellement modifier la verbosité des logs dans la configuration

### Est-ce un bug ?

-   Si le client CLI crash, c’est un bug
-   Si le dashboard web ou une API renvoie une erreur 500, c’est peut-être un bug
-   Si les logs montrent une stacktrace Python, c’est un bug
-   Sinon, à vous d’en juger

## Opérations

### Gestion des logs

-   Centraliser les logs
-   Logs d'API
-   Logs autres composants OpenStack
-   Logs BDD, AMQP, etc.

### Backup

-   Bases de données
-   Mécanisme de déploiement, plutôt que les fichiers de configuration

### Monitoring

-   Réponse des APIs
-   Vérification des services OpenStack et dépendances

### Utilisation des quotas

-   Limiter le nombre de ressources allouables
-   Par utilisateur ou par projet
-   Support dans Nova
-   Support dans Cinder
-   Support dans Neutron

