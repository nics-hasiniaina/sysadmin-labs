# Configurer un routage inter-vlan en utilisant une seule interface du routeur

## Objectif
- [x] Faire communiquer plusieurs VLANs
- [x] Comprendre le trunk et sa configuration
- [x] Savoir configurer des sous-interfaces dans un routeur cisco

## Environnements
- Simulateur: Cisco Packet Tracer
- Machines

|Addresses IP|Masques|Passerelle|Port Switch|
|------------|-------|----------|-----------|
|192.168.10.10|255.255.255.0|192.168.10.1|fa0/1|
|192.168.20.20|255.255.255.0|192.168.20.1|fa0/13|

- Switch

|VLANs|Ports|
|-----|-----|
|VLAN 10|fa0/1 - 12|
|VLAN 20|fa0/13 - 24|
|Trunk|gi0/1|

- Routeur (interface Gi0/0)

|Sous-interfaces|Addresses IP|Masques|Tag|
|---------------|------------|-------|---|
|gi0/0.10|192.168.10.1|255.255.255.0|dot1q 10|
|gi0/0.20|192.168.20.1|255.255.255.0|dot1q 20|

## Étapes de configuration
> [!NOTE]
> Étape 1: Configurer les VLANs [_voir les détails_](https://github.com/nics-hasiniaina/sysadmin-labs/tree/main/networking/configs/VLAN)

> [!NOTE]
> Étape 2: Configurer le **Trunk**

- Sélectionner l'interface, activer le mode **Trunk**, autauriser les VLANs
```cisco
    Switch (config)# interface gi0/1
    Switch (config-if)# switchport mode trunk
    Switch (config-if)# switchport allowed vlan 10,20
```

- Vérification
```cisco
    Switch# show interface trunk
    Port        Mode         Encapsulation  Status        Native vlan
    Gig0/1      on           802.1q         trunking      1

    Port        Vlans allowed on trunk
    Gig0/1      10,20

    Port        Vlans allowed and active in management domain
    Gig0/1      10,20

    Port        Vlans in spanning tree forwarding state and not pruned
    Gig0/1      10,20

    Switch# 
```
> [!WARNING]
> **Résultat :** 
> Le mode trunk, l'encapsulation dot1q sont active, l'interface est maintentant configurée en trunk (`status trunking`), et les VLANs autorisés sont les VLANs 10 et 20.

- Sauvergarde
```cisco
    Switch# write memory
```

> [!NOTE]
> Étape 3: Configurer les sous-interfaces du routeur

- [x] Sélectionner une interface puis démarrer
```cisco
    Router (config)# interface gi0/0
    Router (config-if)# no shutdown    
```

- [x] Sélectionnner les sous-interfaces, activer le tagging (IEEE 802.1q), assigner une addresse IP
```cisco
    Router (config)# interface gi0/0.10
    Router (config-subif)# encapsulation dot1q 10
    Router (config-subif)# ip address 192.168.10.1 255.255.255.0 
    Router (config-subif)# exit

    Router (config)# interface gi0/0.20
    Router (config-subif)# encapsulation dot1q 20
    Router (config-subif)# ip address 192.168.20.1 255.255.255.0
    Router (Config-subif)# exit
```
- [x] Vérifier les addresses IP
```cisco
    Router# show ip route
         192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
    C       192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
    L       192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
        192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
    C       192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
    L       192.168.20.1/32 is directly connected, GigabitEthernet0/0.20

    Router#
```
> [!WARNING]
> **Résultat :** 
> Les addresses IP sont bien assigner aux sous-interfaces gi0/0.10 et gi0/0.20

- [x] Sauvegarder avec ```write memory```

## Tests
### 1°/ Pinguer les PCs depuis le routeur
```cisco
    Router# ping 192.168.10.10

    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 192.168.10.10, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

    Router#ping 192.168.20.20

    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 192.168.20.20, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

    Router#
```
> [!WARNING]
> **Résultat :** 
> On a '_Success rate is 100 percent (5/5)_'; ça prouve que la connexion entre les PCs et le routeur fonctionne correctement (trunk, sous-interfaces, tagging, passerelle)

### 2°/ Ping PC1 vers PC2
```
    C:\>ping 192.168.20.20

    Pinging 192.168.20.20 with 32 bytes of data:

    Reply from 192.168.20.20: bytes=32 time<1ms TTL=127
    Reply from 192.168.20.20: bytes=32 time<1ms TTL=127
    Reply from 192.168.20.20: bytes=32 time=15ms TTL=127
    Reply from 192.168.20.20: bytes=32 time<1ms TTL=127

    Ping statistics for 192.168.20.20:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 0ms, Maximum = 15ms, Average = 3ms

    C:\>
```
> [!WARNING]
> **Résultat :** 
> On a '_lost = 0 (0% loss)_'; ça montre que PC1 et PC2, situés dans deux VLANs différents se communiquent sans problème.

## Commandes clés
|Commandes|Utilisations|
|---------|------------|
|`switchport mode trunk`|Activer le mode trunk|
|`switchport trunk encapsulation dot1q`|Activation du tagging 802.1q (Obligatoire pour les anciens Switch)|
|`switchport trunk allowed vlan <num_vlan>`|Spécifier les VLANs pouvant passer sur le trunk|
|`no shutdown`|Démarrer une interface|
|`interface <nom_interface>0/<num_interface>.<num_vlan>`|Sélectionner une sous-interface (ex: `interface gi0.0.1`0)|
|`encapsulation dot1q <num_vlan>`|Active l'ajout de l'etiquette <num_vlan> dans les paquets passant par cette interface (ex: `encapsulation dot1q 10`)|

> [!NOTE]
> **Conclusion :** 
> Ce lab m'a appris beacoup de chose tels que: la configuration des sous-interfaces, le trunk, le fonctionnement du tag 802.1q, ...