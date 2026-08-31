# Guide de configuration — Réseau Impact Influence

> Environnement de laboratoire (Cisco Packet Tracer). Ce guide résume les grandes étapes de configuration ; les commandes complètes par équipement sont disponibles dans les fichiers de configuration du projet.

## 1. Préparation de la maquette

- Modèles recommandés : routeurs RED en Cisco 2911-24PS (Layer 3, HSRP, LACP, NAT), routeurs BLUE/GREEN en Cisco 4331, switches de distribution en Cisco 3650-24PS (Layer 3, HSRP, LACP), switches d'accès en Cisco 2960.
- Ajout de modules NIM-1GE-CU-SFP sur les routeurs nécessitant plus de 2 liaisons WAN.
- Câblage automatique (câble éclair) recommandé pour laisser Packet Tracer choisir le bon type de liaison.

## 2. Configuration des routeurs WAN (OSPF)

Chaque routeur WAN (WAN1 à WAN5) est configuré avec :

- Des interfaces routées point-à-point (`no switchport`, `ip ospf network point-to-point`).
- Un processus OSPF unique (area 0) avec un router-id dédié par équipement.
- Les liaisons vers les bâtiments distants (BLUE, GREEN) sont déclarées en `passive-interface` côté WAN, car ces sites n'ont pas besoin de recevoir les annonces OSPF vers leur LAN interne.

## 3. Configuration des routeurs de sortie (RED-1, RED-2)

- Interfaces WAN en NAT outside, interfaces vers D1/D2 en NAT inside.
- Route par défaut principale + route de secours (floating static route, distance administrative 10) vers l'autre routeur RED en cas de panne.
- Routes statiques réparties par VLAN : RED-1 priorise D1 pour DNS/IT/Finance et D2 pour Commercial/RH/Direction (avec bascule croisee en floating route).
- NAT overload (PAT) : une ACL standard autorise les 6 sous-réseaux VLAN à sortir via l'interface WAN en `overload`.

## 4. Configuration des switches de distribution (D1, D2)

- Création des VLAN 10 à 60 (nommés par service) + VLAN 99 natif.
- **EtherChannel LACP** : 4 interfaces physiques regroupées en `Port-channel1` (mode `active` sur D1, `passive` sur D2), configuré en trunk pour tous les VLAN.
- **Interfaces VLAN (SVI)** avec adresse IP réelle par switch + adresse virtuelle HSRP partagée (`standby X ip ...`), priorité HSRP différenciée pour répartir le rôle actif (D1 prioritaire sur 10/20/30, D2 prioritaire sur 40/50/60) et `preempt` activé pour la reprise automatique.
- **Spanning Tree (Rapid-PVST)** : priorité de racine différenciée par groupe de VLAN, pour que D1 et D2 se partagent le rôle de racine.
- **DHCP** : un pool par VLAN, avec exclusion des adresses basses (infrastructure) et hautes (passerelles).
- **ACL étendues** : une ACL par VLAN, appliquée en entrée sur chaque SVI, autorisant les flux applicatifs standards et l'ICMP selon la logique décrite dans le schéma des flux.

## 5. Configuration des switches d'accès

Chaque switch d'accès est configuré avec :

- Son VLAN métier dédié + VLAN natif 99.
- Deux ports trunk 802.1Q vers D1 et D2 (redondance de chemin).
- Un ou plusieurs ports d'accès vers les postes de travail, associés au VLAN du service.

## 6. Configuration des routeurs BLUE et GREEN

- Interface WAN en NAT outside (adresse publique fournie par l'opérateur WAN).
- Interface LAN en NAT inside, vers le serveur applicatif (Mail ou Web).
- Route par défaut vers le nœud WAN correspondant.
- NAT statique : traduction 1:1 de l'adresse privée du serveur vers l'adresse publique du routeur.

## 7. Configuration des serveurs applicatifs

| Serveur | Adresse | Services activés |
|---|---|---|
| SRV-DNS | 192.168.10.10 | DNS (enregistrements A pour le domaine et les sous-domaines mail) |
| SRV-WEB | 192.168.200.10 | HTTP, HTTPS |
| SRV-MAIL | 192.168.201.10 | SMTP, POP3 |

Le serveur DNS interne héberge la résolution du domaine public (`impactinfluence.com` → IP publique du serveur Web) ainsi que les sous-domaines de messagerie.

## 8. Vérifications de base après déploiement

```
show ip interface brief          # état des interfaces
show ip ospf neighbor            # adjacences OSPF établies
show etherchannel summary        # état du Port-channel LACP (Po1 SU, 4 ports P)
show standby brief               # répartition du rôle actif/standby HSRP
show spanning-tree vlan <id>     # racine STP par VLAN
show ip route                    # table de routage et routes de secours
```
