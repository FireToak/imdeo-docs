# Procédure - Réinitialisation d'un commutateur Cisco 2960

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---
## Informations générales

* **Créateur :** Louis MEDO
* **Date de création :** 26/03/2026
* **Dernière modification :** 26/03/2026
* **Validation technique :** Validé par Louis MEDO

---
## Sommaire

- A. Suppression de la configuration initiale
- B. Suppression de la base de données des VLAN
- C. Redémarrage de l'équipement

---
## A. Suppression de la configuration initiale

> Cette étape supprime le fichier de configuration principal chargé au démarrage du switch afin de repartir d'une page blanche.

1. **Passage en mode privilégié et effacement de la NVRAM.**

    ```CISCO
    ! Accès au mode privilégié
    switch> enable

    ! Suppression de la configuration de démarrage
    switch# erase startup-config
    ```

    **Explication des commandes :**

    `enable` : Permet de passer du mode utilisateur (restreint) au mode d'exécution privilégié (accès aux commandes d'administration).
    
    `erase startup-config` : Efface le fichier de configuration sauvegardé dans la mémoire NVRAM. *Note : Il faudra appuyer sur `Entrée` pour confirmer l'action.*

---
## B. Suppression de la base de données des VLAN

> Sur les switchs Cisco, la configuration des VLAN (et du VTP) n'est pas stockée dans le fichier `startup-config`, mais dans un fichier distinct situé dans la mémoire flash. Il est impératif de le supprimer pour une remise à zéro totale.

1. **Suppression du fichier flash.**

    ```CISCO
    ! Suppression du fichier vlan.dat
    switch# delete vlan.dat
    ```

    **Explication de la commande :**

    `delete vlan.dat` : Supprime définitivement le fichier contenant la base de données des VLAN. *Note : Le système demandera deux confirmations consécutives, appuyez sur `Entrée` à chaque fois.*

---
## C. Redémarrage de l'équipement

> Pour que la suppression de la configuration et des VLAN soit prise en compte, le commutateur doit être redémarré.

1. **Reboot du switch.**

    ```CISCO
    ! Redémarrage
    switch# reload
    ```

    **Explication de la commande :**

    `reload` : Lance le processus de redémarrage de l'IOS Cisco. 

    **⚠️ Attention :** Si le switch vous demande de sauvegarder la configuration actuelle avant de redémarrer (*"System configuration has been modified. Save?"*), vous devez absolument taper **`no`**, sinon vous annulerez tout le travail de réinitialisation.

---
## Annexe

- [Procédure - Configuration initiale d'un commutateur Cisco](https://www.cisco.com/c/fr_ca/support/docs/smb/switches/cisco-350-series-managed-switches/smb5559-how-to-manually-reload-or-reset-a-switch-through-the-command.html)