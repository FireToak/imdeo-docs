# Procédure - Réinitialisation d'un routeur Cisco 1921

**Situation : Remise à zéro complète d'un routeur de l'infrastructure avant un nouveau déploiement.**

**Contexte : IMDEO**

![](https://ap-bts-sio-louis.github.io/imdeo/assets/logo_imdeo.jpg)

---
## Informations générales

* **Créateur :** Louis MEDO
* **Date de création :** 26/03/2026
* **Dernière modification :** 26/03/2026
* **Validation technique :** Validé par Louis MEDO

---
## Sommaire

- A. Suppression de la configuration initiale
- B. Suppression des clés cryptographiques (RSA)
- C. Redémarrage du routeur

---
## A. Suppression de la configuration initiale

> Cette étape supprime le fichier de configuration principal chargé au démarrage du routeur.

1. **Passage en mode privilégié et effacement de la NVRAM.**

    ```CISCO
    ! Accès au mode privilégié
    router> enable

    ! Suppression de la configuration de démarrage
    router# erase startup-config
    ```

    **Explications :**

    `enable` : Élève les privilèges de l'utilisateur pour passer du mode restreint au mode d'administration.

    `erase startup-config` : Vide la mémoire NVRAM en supprimant le fichier de configuration de démarrage. *Validation requise en appuyant sur `Entrée`.*

---
## B. Suppression des clés cryptographiques (RSA)

> Contrairement à un switch standard, un routeur possède souvent des clés de chiffrement générées pour les accès distants sécurisés (SSH) ou les tunnels VPN. Il est recommandé de les purger.

1. **Passage en mode configuration globale et suppression des clés.**

    ```CISCO
    ! Accès au mode de configuration globale
    router# configure terminal

    ! Suppression des clés RSA
    router(config)# crypto key zeroize rsa

    ! Retour au mode privilégié
    router(config)# exit
    ```

    **Explications :**

    `configure terminal` (ou `conf t`) : Permet d'accéder au mode de configuration globale pour modifier les paramètres de l'équipement.

    `crypto key zeroize rsa` : Détruit toutes les clés de chiffrement RSA stockées sur le routeur. *Confirmer avec `yes` si demandé.*

    `exit` : Remonte d'un niveau dans l'arborescence des modes de commande Cisco (retour au mode privilégié).

---
## C. Redémarrage du routeur

> Le redémarrage est obligatoire pour vider la configuration actuellement active en RAM et redémarrer sur une mémoire vierge.

1. **Reboot de l'équipement.**

    ```CISCO
    ! Redémarrage
    router# reload
    ```

    **Explications :**
    `reload` : Déclenche la séquence de redémarrage du système d'exploitation (Cisco IOS).

    **⚠️ Attention :** Si le routeur propose de sauvegarder la configuration modifiée (*"System configuration has been modified. Save?"*), il faut impérativement répondre **`no`**.