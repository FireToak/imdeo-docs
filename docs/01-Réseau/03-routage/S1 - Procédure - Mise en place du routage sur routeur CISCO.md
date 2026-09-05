# S1 - Procédure - Mise en place du routage sur routeur CISCO

**Situation 1 : Préparation de l'infrastructure réseau d'IMDEO : Adressage et maquettage**

**Contexte : IMDEO**

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---
## Informations

- **Créateur :** Louis MEDO
- **Date de création :** 15/01/2026
- **Validation technique :**

---
## Sommaire

1. Configuration des interfaces connectées avec l'encapsulation dot1Q.
2. Configuration de l'interface de sortie WAN.
3. Configuration de la route par défaut.

---
## 1. Configuration des interfaces connectées (dot1Q)

Pour permettre le routage inter-VLAN depuis un routeur (méthode dite "Router-on-a-Stick"), il faut activer l'interface physique, puis créer des sous-interfaces logiques pour chaque VLAN.

```Cisco
enable
configure terminal
interface gigabitEthernet 0/1
no shutdown
exit
```

`enable` / `configure terminal` : Passage en mode d'exécution privilégié puis en mode de configuration globale.

`interface gigabitEthernet 0/1` : Sélectionne l'interface physique principale reliée au commutateur.

`no shutdown` : Allume électriquement le port physique (étape indispensable pour que les sous-interfaces fonctionnent).

**Création de la sous-interface (Exemple pour le VLAN 10) :**

```Cisco
interface gigabitEthernet 0/1.10
encapsulation dot1Q 10
ip address 192.168.10.254 255.255.255.0
exit
```

`interface gigabitEthernet 0/1.10` : Crée et accède à la sous-interface logique `.10` (par bonne pratique, on utilise le numéro du VLAN comme identifiant).

`encapsulation dot1Q 10` : Active le standard 802.1Q sur cette sous-interface pour qu'elle lise et tague les trames destinées au VLAN 10.

`ip address ...` : Assigne l'adresse IP qui servira de passerelle par défaut pour les équipements de ce VLAN.

---
## 2. Configuration de l'interface de sortie WAN

Il s'agit de l'interface physique orientée vers l'extérieur (routeur de bordure ou FAI).

```Cisco
interface gigabitEthernet 0/0
ip address 172.16.31.1 255.255.252.0
no shutdown
exit
```

`interface gigabitEthernet 0/0` : Accède à l'interface physique WAN.

`ip address 172.16.31.1 255.255.252.0` : Attribue l'adresse IP d'interconnexion (souvent fournie par le FAI ou définie dans le plan d'adressage externe).

`no shutdown` : Active l'interface.

---
## 3. Configuration de la route par défaut

La route par défaut permet d'envoyer tout le trafic externe (Internet ou réseaux distants) vers le routeur suivant, lorsque la destination n'est pas connue dans la table de routage locale.

```Cisco
ip route 0.0.0.0 0.0.0.0 172.16.31.254
```

`ip route` : Commande de création d'une route statique.

`0.0.0.0 0.0.0.0` : Représente "tous les réseaux de destination" avec "n'importe quel masque de sous-réseau" (définition de la route par défaut).

`172.16.31.254` : Adresse IP de passerelle du prochain saut (l'équipement du FAI recevant le trafic).

---
## Ressources

- [Documentation CISCO - Configuration du routage inter-VLAN](https://www.cisco.com/c/fr_ca/support/docs/lan-switching/inter-vlan-routing/14976-50.html)