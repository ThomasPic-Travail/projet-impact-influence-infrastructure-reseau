# Schéma physique — Réseau Impact Influence

## Topologie générale

Topologie hiérarchique à trois couches, répartie sur trois bâtiments (BLUE, GREEN, RED), interconnectés par un réseau WAN interne à 5 nœuds fonctionnant en OSPF (area 0).

## Réseau WAN

| Équipement | Rôle | Type |
|---|---|---|
| WAN1 à WAN5 | Commutateurs multicouches, adjacences OSPF area 0 | Cisco 3650 |

Les liens entre les 5 nœuds WAN sont des adjacences de routage (pas de VLAN ni de port applicatif) : le WAN transporte uniquement du trafic OSPF et le trafic routé entre sites.

## Bâtiment BLUE (messagerie)

- **Routeur BLUE** (Cisco 2911) : interface WAN vers WAN3 (adresse publique 132.186.32.74), interface LAN vers le serveur de messagerie.
- **SRV-MAIL** : 192.168.201.10 — services SMTP et POP3 actifs.
- NAT statique : IP privée du serveur Mail traduite vers l'IP publique du routeur BLUE.

## Bâtiment GREEN (serveur Web)

- **Routeur GREEN** (Cisco 2911) : interface WAN vers WAN5 (adresse publique 132.186.32.82), interface LAN vers le serveur Web.
- **SRV-WEB** : 192.168.200.10 — services HTTP et HTTPS actifs.
- NAT statique : IP privée du serveur Web traduite vers l'IP publique du routeur GREEN.

## Bâtiment RED (site principal)

### Routeurs de sortie

- **RED-1** et **RED-2** (Cisco 2911, avec module NAT) : deux routeurs indépendants, sans interconnexion directe entre eux, chacun relié à D1 et D2 via des liens routés point-à-point (/30).
  - RED-1 : Gi0/0 → WAN1, Gi0/1 → D1, Gi0/2 → D2
  - RED-2 : Gi0/0 → WAN2, Gi0/1 → D1, Gi0/2 → D2

### Switches de distribution

- **D1** (Cisco 3650) : actif HSRP sur les VLAN 10/20/30, racine STP pour ces VLAN, serveur DHCP, gestion des ACL.
- **D2** (Cisco 3650) : actif HSRP sur les VLAN 40/50/60, racine STP pour ces VLAN, serveur DHCP, gestion des ACL.
- **Po1 (EtherChannel LACP)** : 4 liens agrégés entre D1 et D2 (Gi1/0/3 à Gi1/0/6), configurés en trunk 802.1Q pour les VLAN 10 à 60 et le VLAN natif 99.

### Switches d'accès

Chaque VLAN métier dispose de son propre switch d'accès (Cisco 2960), relié en trunk 802.1Q aux deux switches de distribution (lien principal choisi par STP, lien redondant activé automatiquement en cas de panne) :

| Switch d'accès | VLAN | Port de distribution (D1/D2) |
|---|---|---|
| SW-DNS | 10 | Gi1/0/7 |
| SW-IT | 20 | Gi1/0/8 |
| SW-FINANCE | 30 | Gi1/0/9 |
| SW-COMMERCIAL | 40 | Gi1/0/10 |
| SW-RH | 50 | Gi1/0/11 |
| SW-DIRECTION | 60 | Gi1/0/12 |

## Types de liaisons utilisées

| Type | Usage |
|---|---|
| Trunk 802.1Q | VLAN 10-60 + 99, liens principaux et redondants (switches d'accès ↔ distribution) |
| Lien routé point-à-point (/30) | Liaisons avec adresse IP dédiée (RED-1/RED-2 ↔ D1/D2, WAN inter-sites) |
| EtherChannel LACP | Po1, 4 liens agrégés entre D1 et D2 |
| Lien WAN OSPF | Adjacences de routage dynamique entre les 5 nœuds WAN |
| Lien d'accès | Connexion PC / serveur au switch d'accès |

## Équipements utilisés

- Routeur Cisco 2911 : RED-1, RED-2, BLUE, GREEN.
- Switch L3 Cisco 3650 : WAN1 à WAN5, D1, D2.
- Switch d'accès Cisco 2960 : SW-DNS à SW-DIRECTION.
- Serveurs : DNS, WEB, MAIL.
- Postes de travail (PC) par service métier.
