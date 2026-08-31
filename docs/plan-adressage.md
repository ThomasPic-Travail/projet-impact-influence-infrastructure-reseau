# Plan d'adressage IP — Réseau Impact Influence

## Bâtiment RED — VLAN métier

| VLAN | Usage | Sous-réseau | Masque | Première IP utilisable | Dernière IP utilisable | Broadcast |
|---|---|---|---|---|---|---|
| 10 | DNS | 192.168.10.0/24 | 255.255.255.0 | 192.168.10.1 | 192.168.10.254 | 192.168.10.255 |
| 20 | IT | 192.168.20.0/24 | 255.255.255.0 | 192.168.20.1 | 192.168.20.254 | 192.168.20.255 |
| 30 | Finance | 192.168.30.0/24 | 255.255.255.0 | 192.168.30.1 | 192.168.30.254 | 192.168.30.255 |
| 40 | Commercial | 192.168.40.0/24 | 255.255.255.0 | 192.168.40.1 | 192.168.40.254 | 192.168.40.255 |
| 50 | RH | 192.168.50.0/24 | 255.255.255.0 | 192.168.50.1 | 192.168.50.254 | 192.168.50.255 |
| 60 | Direction | 192.168.60.0/24 | 255.255.255.0 | 192.168.60.1 | 192.168.60.254 | 192.168.60.255 |
| 99 | Natif / Management | — | — | — | — | — |

## Bâtiment RED — Liens point-à-point (D1/D2 ↔ RED-1/RED-2)

| Liaison | Sous-réseau | Masque |
|---|---|---|
| D1 ↔ RED-1 | 192.168.100.0/30 | 255.255.255.252 |
| D1 ↔ RED-2 | 192.168.100.4/30 | 255.255.255.252 |
| D2 ↔ RED-1 | 192.168.100.8/30 | 255.255.255.252 |
| D2 ↔ RED-2 | 192.168.100.12/30 | 255.255.255.252 |

## Bâtiment BLUE

| Élément | Sous-réseau / IP | Masque |
|---|---|---|
| LAN BLUE | 192.168.201.0/24 | 255.255.255.0 |
| Liaison WAN3 ↔ BLUE | 132.186.32.72/30 | 255.255.255.252 |
| Serveur MAIL (privé) | 192.168.201.10 | — |
| Routeur BLUE (public) | 132.186.32.74 | — |

## Bâtiment GREEN

| Élément | Sous-réseau / IP | Masque |
|---|---|---|
| LAN GREEN | 192.168.200.0/24 | 255.255.255.0 |
| Liaison WAN5 ↔ GREEN | 132.186.32.80/30 | 255.255.255.252 |
| Serveur WEB (privé) | 192.168.200.10 | — |
| Routeur GREEN (public) | 132.186.32.82 | — |

## Adresses réservées

| Rôle | Adresse |
|---|---|
| Serveur DNS | 192.168.10.10 |
| Serveur WEB | 192.168.200.10 |
| Serveur MAIL | 192.168.201.10 |
| Passerelle virtuelle HSRP | 192.168.X.254/24 (une par VLAN) |

## Répartition DHCP par VLAN (postes utilisateurs)

| VLAN | Plage DHCP attendue | Passerelle |
|---|---|---|
| IT (20) | 192.168.20.51 → .251 | 192.168.20.254 |
| Finance (30) | 192.168.30.51 → .251 | 192.168.30.254 |
| Commercial (40) | 192.168.40.51 → .251 | 192.168.40.254 |
| RH (50) | 192.168.50.51 → .251 | 192.168.50.254 |
| Direction (60) | 192.168.60.51 → .251 | 192.168.60.254 |

Les adresses `.1` à `.50` et `.252` à `.254` de chaque plage sont exclues du pool DHCP (réservées à l'infrastructure et aux passerelles).
