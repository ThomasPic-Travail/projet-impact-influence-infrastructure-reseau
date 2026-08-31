# Schéma logique des flux — Réseau Impact Influence

## Vue d'ensemble

Ce document détaille les flux applicatifs autorisés entre les différents segments du réseau : protocoles, ports, sens de communication et justification métier.

## Tableau récapitulatif des flux

| # | Flux / Application | Protocole | Port | Couche | Source | Destination | Sens |
|---|---|---|---|---|---|---|---|
| 1 | Test de connectivité (ping) | ICMP | — | L3 | PC-IT (VLAN 20) | Tous VLAN | Bidirectionnel |
| 2 | Résolution DNS | DNS | 53 | UDP/TCP | Tous les PC | SRV-DNS 192.168.10.10 | Requête / Réponse |
| 3 | Navigation Web HTTP | HTTP | 80 | TCP | Tous les PC | SRV-WEB (via NAT GREEN) | Requête / Réponse |
| 4 | Navigation Web sécurisée | HTTPS | 443 | TCP | Tous les PC | SRV-WEB (via NAT GREEN) | Requête / Réponse |
| 5 | Envoi de mail | SMTP | 25 | TCP | Client mail | SRV-MAIL (via NAT BLUE) | Envoi |
| 6 | Réception de mail | IMAP | 143 | TCP | Client mail | SRV-MAIL (via NAT BLUE) | Réception |
| 7 | Attribution IP | DHCP | 67/68 | UDP | D1 / D2 | Tous les PC | Offre DHCP |
| 8 | Redondance passerelle | HSRP | 1985 | UDP | D1 | D2 | Bidirectionnel (Hello) |
| 9 | Routage dynamique | OSPF | — | L3 | D1, D2, RED-1, RED-2, WAN1-5 | Tous les routeurs L3 | Bidirectionnel |
| 10 | Synchronisation horloge | NTP | 123 | UDP | RED-1 / RED-2 (maître) | D1, D2, switches | Réponse NTP |
| 11 | Administration sécurisée | SSH | 22 | TCP | Admin (VLAN 99) | Tous les équipements | Connexion admin |
| 12 | Agrégation de liens | LACP | — | L2 | D1 | D2 | Négociation EtherChannel |
| 13 | Spanning Tree | STP/RSTP | — | L2 | D1 / D2 | Switches d'accès | BPDU |

## Logique de filtrage par VLAN

Le filtrage (ACL) appliqué sur D1 et D2 suit une règle constante quel que soit le VLAN :

- **Autorisé pour tous les VLAN** : DNS (53), HTTP (80), HTTPS (443), SMTP (25), POP3 (110), DHCP (67/68), HSRP (1985 vers l'adresse multicast 224.0.0.2).
- **VLAN IT uniquement** : ICMP initié vers n'importe quelle destination (rôle de diagnostic réseau).
- **Autres VLAN** : ICMP en écho-réponse uniquement — ils peuvent répondre à un ping mais ne peuvent pas en initier vers d'autres VLAN.
- **Reste du trafic** : refusé implicitement (`deny ip any any` en fin de liste).

Cette logique permet à l'équipe IT de diagnostiquer l'ensemble du réseau depuis son VLAN, tout en empêchant les autres services de sonder le réseau entre eux.

## Zone WAN

Les 5 nœuds WAN (Cisco 3650) échangent uniquement du trafic de routage OSPF (area 0) — aucun port applicatif n'y transite directement, ce sont des points de transit purs entre les bâtiments RED, BLUE et GREEN.
