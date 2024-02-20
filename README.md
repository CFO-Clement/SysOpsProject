# SysOpsProject
## Logistique
#### Objectif :
L'objectif de ce pipeline est de créer un système d'intégration continue pour un site de papeterie. Il vise à automatiser les étapes de construction (build) et de déploiement du site, afin d'améliorer l'efficacité et la fiabilité des mises à jour.

#### Solution :
Nous proposons de créer un pipeline automatisé qui prendra en charge les différentes étapes de build et de déploiement. Ce processus sera déclenché automatiquement par les commits sur le repository, assurant ainsi une mise à jour continue et fluide du site de papeterie.

#### Workflow :
1. Un membre de l'équipe DevOps crée une nouvelle version du site, de l'API, ou modifie le schéma de la base de données.
2. Une fois cette fonctionnalité testée et approuvée, elle est soumise via une Pull Request (PR). Cette PR doit être validée par d'autres membres de l'équipe.
3. La validation de la PR sur la branche principale (main) déclenche automatiquement le pipeline.
4. Le pipeline exécute alors les opérations suivantes :
   a. Récupération du code depuis le repository.
   b. Construction (build) des différentes images Docker nécessaires.
   c. Publication (push) des images Docker sur un registre sécurisé.
   d. Notification au cluster Kubernetes (k8s) pour qu'il se mette à jour avec la dernière version disponible sur le registre Docker.

#### Questionnements ouverts :
- Quelles sont les étapes spécifiques de votre processus de build ? Nous avons compris que vous utilisez actuellement un simple `yarn start` sans créer d'artefact préalablement (par exemple, avec `yarn build`). Pourriez-vous clarifier cette approche ?
- Le fichier `docker-compose.yml` est-il susceptible d'évoluer fréquemment, ou pouvons-nous le considérer comme stable ? Cette information nous aidera à anticiper les besoins de maintenance et d'adaptation du pipeline.

La mise en place et la mise à jour d'un cluster en utilisant Ansible, ainsi que l'utilisation de runners GitHub pour gérer un pipeline d'intégration et de déploiement continu, impliquent plusieurs étapes techniques détaillées. Voici un aperçu de ces étapes :


## Technique
#### 1. Préparation de l'environnement Ansible

- **Installation d'Ansible :** Assurez-vous qu'Ansible est installé sur la machine qui agira comme le contrôleur Ansible.
- **Configuration de l'inventaire :** Créez un fichier d'inventaire Ansible (`hosts` par exemple) pour lister toutes les machines qui feront partie de votre cluster, en spécifiant leurs adresses IP et éventuellement des groupes (comme `masters` et `nodes` pour un cluster Kubernetes).

#### 2. Création du cluster avec Ansible

- **Rédaction des Playbooks Ansible :** Écrivez des playbooks Ansible qui définiront les tâches nécessaires à l'exécution sur les machines cibles pour :
  - Installer les prérequis (comme Docker, kubeadm, kubelet, et kubectl sur chaque machine pour un cluster Kubernetes).
  - Initialiser le cluster sur le(s) master(s) en utilisant `kubeadm init` ou un outil similaire.
  - (Joindre les nœuds workers au cluster avec `kubeadm join` ou une commande équivalente.)
  - (Configurer le réseau du cluster en déployant un CNI (Container Network Interface) comme Calico ou Weave.)
- **Exécution des Playbooks :** Lancez les playbooks Ansible pour automatiser le déploiement et la configuration du cluster sur les machines cibles.

#### 3. Mise à jour du cluster avec Ansible

- **Playbooks de mise à jour :** Créez ou ajustez vos playbooks pour inclure des tâches de mise à jour du cluster, comme la mise à jour de la version des composants Kubernetes sur tous les nœuds.
- **Exécution des mises à jour :** Utilisez Ansible pour exécuter ces playbooks et mettre à jour le cluster en fonction des besoins.

#### 4. Configuration des runners GitHub pour la pipeline

- **Création de runners GitHub :** Configurez des self-hosted runners GitHub dans votre infrastructure ou utilisez les runners hébergés par GitHub pour exécuter vos pipelines.
- **Définition de la pipeline CI/CD :** Dans votre dépôt GitHub, créez un fichier `.github/workflows/ci-cd.yml` pour définir les étapes de votre pipeline d'intégration et de déploiement continu, qui peuvent inclure :
  - Build de l'application.
  - (Tests unitaires et d'intégration.)
  - Déploiement de l'application sur le cluster via Ansible.

#### 5. Automatisation et intégration

- **Intégration Ansible avec GitHub Actions :** Utilisez les actions GitHub pour exécuter des commandes Ansible directement depuis votre pipeline CI/CD, permettant ainsi le déploiement automatique des mises à jour de votre application sur le cluster.
- (**Gestion des secrets :** Configurez les secrets dans GitHub Actions pour stocker en toute sécurité les informations sensibles nécessaires à l'exécution de vos playbooks Ansible et à l'interaction avec votre cluster.)
