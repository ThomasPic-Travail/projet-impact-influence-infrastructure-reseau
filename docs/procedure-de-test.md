# Procédure de test — Réseau Impact Influence

**Auteur** : Thomas Pic
**Outil** : Cisco Packet Tracer

## Contexte

Ce document décrit la procédure de test du réseau du bâtiment RED d'Impact Influence. L'infrastructure repose sur deux routeurs RED-1 et RED-2 (Cisco 2911), deux switches de distribution D1 et D2 (Cisco 3650), des switches d'accès par VLAN, et un réseau WAN simulé avec routeurs OSPF. Les fonctionnalités testées couvrent le routage inter-VLAN, la redondance HSRP, l'agrégation de liens LACP, le NAT, le DNS, la messagerie et l'accès web.

## Prérequis

- Tous les équipements sont démarrés et opérationnels.
- Les PC ont obtenu une adresse IP via DHCP.
- Le serveur DNS (192.168.10.10) est actif avec l'entrée `impactinfluence.com` → 132.186.32.82.
- Le serveur MAIL (BLUE, 192.168.201.10) est actif avec SMTP et POP3 activés.
- Le serveur WEB (GREEN, 192.168.200.10) est actif avec HTTP et HTTPS activés.

## Test 1 — Ping depuis le réseau IT vers les autres services

**Objectif** : vérifier que le VLAN IT peut communiquer avec tous les autres VLAN, conformément aux ACL qui autorisent l'ICMP initié depuis IT uniquement.

**Résultats attendus** :

| Source | Destination | Résultat |
|---|---|---|
| PC IT | Serveur DNS | Réponse 4/4 ✅ |
| PC IT | PC Finance / Commercial / RH / Direction | Réponse 4/4 ✅ |
| PC IT | Serveur MAIL | Réponse 4/4 ✅ |
| PC Finance | PC IT | Échoue (ACL bloque l'ICMP initié hors IT) ❌ |

## Test 2 — Bascule HSRP lors de l'arrêt d'un switch de distribution

**Objectif** : vérifier que HSRP assure la continuité de la passerelle virtuelle lors de la panne d'un switch de distribution.

**Résultats attendus** :

| Étape | Résultat attendu |
|---|---|
| État initial | D1 Active sur VLAN 10/20/30, Standby sur 40/50/60 |
| Après arrêt de D1 | D2 devient Active sur tous les VLAN |
| Ping pendant la bascule | 2 à 4 paquets perdus maximum |
| Ping après bascule | 4/4 réponses via D2 |
| Après rallumage de D1 | D1 reprend le rôle Active sur 10/20/30 (grâce au `preempt`) |

## Test 3 — Suppression de deux liens EtherChannel

**Objectif** : vérifier que le groupe LACP entre D1 et D2 reste opérationnel après la perte de 2 liens sur 4.

**Résultats attendus** :

| Étape | Résultat attendu |
|---|---|
| Avant suppression | Po1(SU) avec 4 ports actifs (P) |
| Après suppression de 2 liens | Po1(SU) avec 2 ports actifs (P) et 2 suspendus (s) |
| Ping inter-VLAN | Trafic toujours actif via les 2 liens restants ✅ |

## Test 4 — Arrêt de RED-1 et continuité du service Internet

**Objectif** : vérifier que l'arrêt de RED-1 ne coupe pas l'accès Internet, grâce à la route de secours via RED-2.

**Résultats attendus** :

| Étape | Résultat attendu |
|---|---|
| Avant arrêt | Route par défaut via RED-1 |
| Après arrêt de RED-1 | Bascule automatique vers RED-2 |
| Ping Internet après bascule | 4/4 réponses via RED-2 ✅ |

## Test 5 — Requêtes DNS depuis les postes

**Objectif** : vérifier la résolution DNS depuis différents VLAN vers le serveur DNS interne.

**Résultat attendu** : `nslookup impactinfluence.com` retourne l'adresse publique du serveur Web (132.186.32.82).

## Test 6 — Envoi et réception de mails

**Objectif** : vérifier l'échange de mails entre services via le serveur de messagerie hébergé à BLUE.

**Résultat attendu** : envoi, réception et réponse fonctionnels entre les comptes de chaque service (Finance, Direction, RH, etc.).

## Test 7 — Accès au serveur Web en HTTP et HTTPS

**Objectif** : vérifier l'accessibilité du serveur Web hébergé à GREEN, par adresse IP et par nom de domaine.

**Résultat attendu** : la page du serveur Web s'affiche correctement en HTTP, en HTTPS, et via le nom de domaine `impactinfluence.com` (résolution DNS + affichage).

## Récapitulatif des tests

| # | Test | Critère de succès |
|---|---|---|
| 1 | Ping IT vers autres VLAN | IT ping tous les VLAN ✅ / Finance ne ping pas IT ❌ |
| 2 | Bascule HSRP (arrêt D1) | D2 devient Active, continuité assurée |
| 3 | EtherChannel — 2 liens supprimés | Po1 reste SU avec 2 ports actifs |
| 4 | Arrêt RED-1 — continuité Internet | Bascule automatique vers RED-2 |
| 5 | Résolution DNS | impactinfluence.com → 132.186.32.82 |
| 6 | Envoi/réception mails | Mail transmis entre utilisateurs |
| 7 | Accès Web HTTP/HTTPS | Page affichée par IP et par nom de domaine |
