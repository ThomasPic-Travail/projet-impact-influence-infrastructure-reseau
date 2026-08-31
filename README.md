# Infrastructure réseau haute disponibilité — Impact Influence

## Contexte

Ce projet consiste à concevoir et déployer l'infrastructure réseau des nouveaux locaux de l'entreprise fictive **Impact Influence**, répartie sur trois bâtiments (RED, BLUE, GREEN), avec un réseau WAN interne simulant l'interconnexion entre sites.

L'objectif est de garantir une infrastructure **robuste et évolutive** : haute disponibilité des passerelles, agrégation de liens, routage dynamique, et exposition sécurisée des services publics (Web, Mail, DNS) tout en cloisonnant les flux internes par service métier.

Environnement de maquettage : Cisco Packet Tracer.

## Architecture

Le réseau s'organise en trois bâtiments :

- **RED** — site principal, structuré en 6 VLAN par service (DNS/IT, IT, Finance, Commercial, RH, Direction), avec deux routeurs (RED-1, RED-2) et deux switches de distribution (D1, D2) redondants.
- **BLUE** — héberge le serveur de messagerie (SMTP/POP3), exposé publiquement via NAT statique.
- **GREEN** — héberge le serveur Web (HTTP/HTTPS), exposé publiquement via NAT statique.

Cinq routeurs/switches WAN (WAN1 à WAN5) interconnectent les trois bâtiments via des adjacences **OSPF** (area 0).

## Fonctionnalités mises en œuvre

- **Segmentation VLAN** : 6 VLAN métier (DNS, IT, Finance, Commercial, RH, Direction) + VLAN natif de management.
- **Haute disponibilité de la passerelle (HSRP)** : D1 et D2 se répartissent le rôle actif par groupe de VLAN, avec bascule automatique en cas de panne (`preempt`).
- **Agrégation de liens (LACP)** : 4 liens physiques agrégés en un unique Port-channel entre D1 et D2, tolérant la perte de plusieurs liens.
- **Spanning Tree (RSTP)** : répartition de la racine entre D1 et D2 selon les VLAN, pour équilibrer la charge et éviter les boucles.
- **Routage dynamique OSPF** : adjacences point-à-point entre les routeurs WAN et les sites distants (BLUE, GREEN), avec routes de secours (floating static routes) entre RED-1 et RED-2.
- **NAT statique** : exposition contrôlée des serveurs Web et Mail vers des adresses publiques dédiées.
- **NAT overload (PAT)** : sortie Internet mutualisée pour les postes utilisateurs du bâtiment RED.
- **DHCP par VLAN** : plages d'adressage dynamique dédiées à chaque service, avec exclusions pour les adresses réservées.
- **ACL avancées par VLAN** : filtrage différencié autorisant DNS/HTTP/HTTPS/SMTP/POP3 pour tous, ICMP initié uniquement depuis le VLAN IT (diagnostic), et blocage explicite du reste.
- **Services applicatifs** : serveur DNS interne (résolution du nom de domaine `impactinfluence.com`), serveur Web, serveur de messagerie avec comptes par service.

## Structure du dépôt

```
├── docs/
│   ├── schema-physique.md          # Topologie physique (équipements, liaisons, adressage inter-sites)
│   ├── schema-logique-flux.md      # Flux applicatifs, protocoles et ports (tableau récapitulatif)
│   ├── plan-adressage.md           # Plan d'adressage IP complet (VLAN, WAN, serveurs)
│   ├── guide-configuration.md      # Configuration pas-à-pas (routeurs, switches, ACL, DHCP, HSRP, LACP)
│   └── procedure-de-test.md        # 7 scénarios de test (bascule HSRP, LACP, panne routeur, DNS, mail, web)
└── README.md
```

## Scénarios de validation

Le projet a été validé par 7 tests couvrant les principaux mécanismes de résilience :

1. Connectivité inter-VLAN conforme aux ACL (IT autorisé partout, autres VLAN restreints).
2. Bascule HSRP lors de l'arrêt d'un switch de distribution (perte de 2 à 4 paquets maximum).
3. Résilience de l'agrégation LACP après suppression de 2 liens sur 4.
4. Continuité de l'accès Internet lors de l'arrêt d'un routeur de sortie (bascule sur route de secours).
5. Résolution DNS interne vers le nom de domaine public.
6. Envoi et réception de mails entre services.
7. Accès au serveur Web en HTTP/HTTPS, par IP et par nom de domaine.

## Auteur

**Thomas Pic** — Formation Administrateur Systèmes, Réseaux et Cybersécurité (Titre RNCP 40356), OpenClassrooms — Projet 7.
