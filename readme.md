# Documentation du contexte IMDEO

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

## Contexte du dépôt

Ce dépôt contient l'ensemble de la documentation technique relative au **contexte IMDEO** dans le cadre du **BLOC 2 - Administration des systèmes et des réseaux** du BTS SIO (Services Informatiques aux Organisations).

Le projet IMDEO (Infrastructure Mutualisée de Déploiement et d'Exploitation Opérationnelle) simule un environnement professionnel d'infrastructure réseau et système comprenant :

- **Services réseau** : DHCP, DNS, routage inter-VLAN, configuration de switchs et routeurs
- **Services système** : Gestion de serveurs Debian, sécurisation du boot, administration Linux
- **Sécurité** : Hardening de systèmes, sécurisation des bootloaders, analyse de vulnérabilités

Cette documentation est **partagée publiquement via GitHub Pages** et construite automatiquement avec **MkDocs Material**.

- **URL du site de documentation :** [Documentation - Infrastructure IMDEO](https://imdeo.bts.loutik.fr)

---

## Organisation du dépôt

Ce dépôt est organisé de façon suivante :

```
bts-sio_imdeo/
├── readme.md # Ce fichier
├── docs_imdeo/ # Dossier principal de la documentation
│ ├── mkdocs.yaml # Configuration MkDocs
│ └── docs/ # Contenu de la documentation
│ ├── 01-Réseau/ # Documentation réseau (DHCP, routage, VLAN...)
│ ├── 02-Système/ # Documentation système (Debian, GRUB, Linux...)
│ └── assets/ # Ressources (images, CSS, logos)
│ ├── css/
│ │ └── style.css # Feuille de style personnalisée
│ └── logo_imdeo.jpg # Logo du projet
```

---

## Utiliser le dépôt

### 📖 Consulter la documentation en ligne

La documentation est disponible publiquement via GitHub Pages :

**🔗 [https://AP-BTS-SIO-Louis.github.io/imdeo/](https://AP-BTS-SIO-Louis.github.io/imdeo/)**

### 🛠️ Lancer la documentation en local

Pour visualiser et modifier la documentation localement :

1. **Cloner le dépôt**
```bash
git clone https://github.com/AP-BTS-SIO-Louis/imdeo.git
cd imdeo/docs_imdeo
```

2. **Installer MkDocs et le thème Material**
```
pip install mkdocs mkdocs-material
```

3. **Lancer le serveur de développement**
```
mkdocs serve
```

4. **Ouvrir dans votre navigateur**
```
http://127.0.0.1:8000
```

## Auteur

**Louis MEDO** | [Linkedin](https://www.linkedin.com/in/louismedo/) | [Portfolio](https://louis.loutik.fr/) | [GitHub](https://github.com/FireToak)