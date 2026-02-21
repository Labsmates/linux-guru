# 🐧 Linux GURU - Guide Complet des Commandes Essentielles

Guide pratique et détaillé des commandes Linux, Docker et OpenShift/Kubernetes pour devenir un expert !

---

## 📚 Contenu

### 1️⃣ [Linux - Commandes Essentielles](./linux-commands.md)
**Tout ce qu'un admin Linux doit connaître :**
- Navigation et gestion de fichiers
- Permissions et ownership
- Processus et monitoring système
- Réseau et diagnostics
- Gestion utilisateurs et groupes
- Archivage et compression
- Recherche et filtrage
- Édition de texte (vim, nano, sed, awk)
- Cron et automatisation
- Package management (apt, yum, dnf)

### 2️⃣ [Docker - Containerisation](./docker-commands.md)
**Maîtriser Docker de A à Z :**
- Images (build, pull, push, tag)
- Containers (run, exec, logs, inspect)
- Volumes (persistance de données)
- Networks (communication entre containers)
- Docker Compose (orchestration multi-containers)
- Dockerfile best practices
- Nettoyage et optimisation
- Troubleshooting et debugging

### 3️⃣ [OpenShift & Kubernetes - Orchestration](./openshift-kubernetes.md)
**Orchestration de containers en production :**
- Pods, Deployments, Services
- ConfigMaps et Secrets
- Volumes persistants (PV/PVC)
- Ingress et Routes (OpenShift)
- Scaling et rolling updates
- Monitoring et logs
- Troubleshooting
- RBAC et sécurité
- Helm charts

### 4️⃣ [CI/CD Pipelines - Automatisation](./cicd-pipelines.md)
**Automatiser builds et déploiements :**
- GitLab CI/CD (pipelines, artifacts, cache)
- GitHub Actions (workflows, matrix, reusable)
- Jenkins (pipelines, shared libraries, JCasC)
- Best practices CI/CD

### 5️⃣ [Terraform - Infrastructure as Code](./terraform.md)
**Gérer l'infrastructure cloud :**
- Syntaxe HCL et providers
- Variables, outputs, locals
- Modules et state management
- AWS resources (EC2, VPC, S3, RDS, ALB)
- Backend S3 + DynamoDB
- Workspaces et environnements

### 6️⃣ [Ansible - Configuration Management](./ansible.md)
**Automatiser la configuration de serveurs :**
- Inventory (static, dynamic)
- Playbooks et roles
- Modules essentiels (apt, copy, template, service)
- Variables et facts
- Loops et conditions
- Ansible Vault (secrets)

### 7️⃣ [Monitoring - Prometheus & Grafana](./monitoring.md)
**Monitorer vos infrastructures :**
- Prometheus (métriques time-series)
- PromQL (query language)
- Alertmanager (notifications)
- Grafana (dashboards)
- Exporters (node, mysql, nginx, blackbox)
- Custom application metrics

---

## 🚀 Démarrage rapide

```bash
# Cloner le repo
git clone https://github.com/Labsmates/linux-guru.git
cd linux-guru

# Consulter un guide
cat linux-commands.md
cat docker-commands.md
cat openshift-kubernetes.md
cat cicd-pipelines.md
cat terraform.md
cat ansible.md
cat monitoring.md
```

---

## 💡 Comment utiliser ce guide

Chaque section contient :
- ✅ **Syntaxe de base** expliquée
- ✅ **Exemples pratiques** commentés
- ✅ **Cas d'usage réels** (scénarios courants)
- ✅ **Astuces et bonnes pratiques**
- ✅ **Commandes avancées** pour aller plus loin

**Format :**
```bash
# Description de la commande
commande [options] argument

# Exemple concret
exemple pratique

# Résultat attendu
output expliqué
```

---

## 📖 Niveau requis

- **Linux :** Débutant à Expert
- **Docker :** Débutant à Intermédiaire
- **OpenShift/K8s :** Intermédiaire à Avancé

---

## 🎯 Objectifs d'apprentissage

Après avoir parcouru ces guides, vous serez capable de :

**Linux :**
- Naviguer efficacement dans un système Linux
- Gérer les permissions et la sécurité
- Automatiser des tâches avec scripts et cron
- Diagnostiquer et résoudre des problèmes système

**Docker :**
- Créer et gérer des containers
- Optimiser vos Dockerfiles
- Orchestrer des applications multi-containers
- Débugger des problèmes de containers

**OpenShift/Kubernetes :**
- Déployer des applications en production
- Gérer le scaling et la haute disponibilité
- Monitorer et debugger des pods
- Sécuriser vos déploiements

**CI/CD :**
- Créer des pipelines automatisés
- Déployer en continu
- Intégrer tests et sécurité
- Gérer artifacts et cache

**Terraform :**
- Définir l'infrastructure as code
- Gérer le state et les backends
- Créer des modules réutilisables
- Déployer multi-cloud

**Ansible :**
- Automatiser la configuration
- Créer des playbooks et roles
- Gérer les inventaires
- Sécuriser avec Vault

**Monitoring :**
- Collecter et visualiser métriques
- Créer dashboards Grafana
- Configurer alertes intelligentes
- Monitorer applications custom

---

## 🛠️ Environnement recommandé

```bash
# Linux
Ubuntu 22.04 LTS / Rocky Linux 9 / Debian 12

# Docker
Docker Engine 24.x + Docker Compose v2

# Kubernetes
Kubernetes 1.28+ / OpenShift 4.14+

# CI/CD
GitLab 16.x / GitHub Actions / Jenkins 2.4x

# IaC & Config Management
Terraform 1.6+ / Ansible 2.15+

# Monitoring
Prometheus 2.45+ / Grafana 10.x
```

---

## 📝 Contribuer

Les contributions sont les bienvenues !

**Comment contribuer :**
1. Fork le repo
2. Créez une branche : `git checkout -b nouvelle-section`
3. Commitez : `git commit -m "Ajout section XYZ"`
4. Push : `git push origin nouvelle-section`
5. Ouvrez une Pull Request

---

## 📄 Licence

MIT License - Libre d'utilisation pour apprentissage et usage professionnel.

---

## 👨‍💻 Auteur

**Wilfrid Peyrius** (Labsmates)  
📅 Février 2026

---

## 🔗 Ressources complémentaires

- [Linux Documentation Project](https://tldp.org/)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [OpenShift Documentation](https://docs.openshift.com/)
- [DevOps Roadmap](https://roadmap.sh/devops)

---

**🌟 Si ce repo vous aide, donnez-lui une étoile sur GitHub !**
