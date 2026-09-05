# S1 - Procédure - Mise en place du NAT CISCO

**Situation 1 : Préparation de l'infrastructure réseau d'IMDEO : Adressage et maquettage**

**Contexte : IMDEO**

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---
## Informations

- **Créateur :** Louis MEDO
- **Date de création :** 21/03/2026
- **Validation technique :**

---
## Sommaire

1. Identification et configuration des interfaces (Inside/Outside).
2. Création de la liste de contrôle d'accès (ACL).
3. Activation de la translation d'adresses (PAT / NAT Overload).
4. Vérification et tests de fonctionnement.

---
## 1. Identification et configuration des interfaces

1. **Définir les rôles des interfaces.** Le routeur doit savoir de quel côté se trouve le réseau privé (Inside) et de quel côté se trouve le réseau public ou externe (Outside) pour appliquer la traduction au bon moment.

    ```cisco
    enable
    configure terminal
    interface gigabitEthernet 0/1
    ip nat inside
    exit
    interface gigabitEthernet 0/0
    ip nat outside
    exit
    ```

    **`ip nat inside`** : Désigne l'interface connectée au réseau local d'IMDEO (réseaux privés à traduire).

    **`ip nat outside`** : Désigne l'interface connectée au FAI ou au réseau externe (réseau public).

-----

## 2. Création de la liste de contrôle d'accès (ACL)

1. **Autoriser les réseaux internes.** L'ACL permet d'identifier précisément quels sous-réseaux ou adresses IP sources ont le droit d'être traduits par le routeur pour accéder à l'extérieur.

    ```cisco
    ! Interdiction d'accès à internet pour le VLAN Administration
    access-list 1 deny 192.168.99.0 0.0.0.255

    ! Autorisation accès internet pour les autres réseaux
    access-list 1 permit 192.168.1.0 0.0.0.255
    access-list 1 permit 172.16.51.0 0.0.0.255
    ```

    **`access-list 1 permit`** : Crée une règle standard (numéro 1) qui autorise le trafic.

    **`access-list 1 deny`** : crée une règle standard qui interdit le trafic.

    **`192.168.1.0 0.0.0.255`** : Spécifie le réseau source et son masque générique (wildcard mask, l'inverse du masque de sous-réseau) qui sera autorisé à sortir.

    **Exemple de configuration pour bloquer un sous réseau administration :**
    
    ```
    access-list 1 deny 192.168.50.64 0.0.0.31
    access-list 1 permit any
    ```

    Il faut bien utiliser le masque inverse du sous réseau avec son adresse réseau ! Surtout le mettre en premier dans les règles de filtrage

-----

## 3. Activation de la translation d'adresses (PAT)

1. **Lier l'ACL à l'interface de sortie.** Cette étape active la traduction d'adresse de port (PAT), permettant à plusieurs machines internes de partager une seule adresse IP publique en utilisant des numéros de ports différents.

    ```cisco
    ip nat inside source list 1 interface gigabitEthernet 0/0 overload
    ```

    **`ip nat inside source list 1`** : Indique au routeur de traduire les adresses sources qui correspondent à l'ACL 1.

    **`interface gigabitEthernet 0/0`** : Définit l'interface (et donc l'adresse IP publique) qui sera utilisée pour remplacer l'adresse source privée.

    **`overload`** : Mot-clé indispensable pour activer le PAT (Port Address Translation). Sans cela, une seule machine pourrait sortir à la fois.

-----

## 4. Vérification et tests de fonctionnement

1. **Générer du trafic.** Depuis un poste client du réseau local (ex: VLAN COM), effectuez un ping vers une adresse externe (ex: 8.8.8.8 ou l'IP du routeur FAI).

    ```cmd
    ping 8.8.8.8
    ```

2. **Consulter la table de traduction.** Sur le routeur, vérifiez que la session NAT a bien été créée et que l'adresse IP privée a été traduite en adresse publique.

    ```cisco
    show ip nat translations
    ```

    **`show ip nat translations`** : Affiche le tableau des correspondances en cours entre les adresses locales (Inside local) et publiques (Inside global).


3. **Vérifier les statistiques globales.** Permet de s'assurer que le routeur intercepte et traduit bien les paquets.

    ```cisco
    show ip nat statistics
    ```

    **`show ip nat statistics`** : Affiche le nombre total de traductions actives, le nombre de "hits" (paquets traduits) et les interfaces configurées pour le NAT.

-----

## Ressources

  - [Documentation CISCO - Configuration du NAT et du PAT](https://www.cisco.com/c/fr_ca/support/docs/security/pix-500-series-security-appliances/15243-19.html)