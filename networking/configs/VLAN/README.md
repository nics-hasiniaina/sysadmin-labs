# Configuration des VLANs dans un switch CISCO

## Objectif:
### - Configurer 03 VLANs pour séparer le traffic des 03 services (RH, IT, PROD) d'une entreprise.
    
## Environnements:
- [x] Simulateur : Cisco Packet Tracer
- [x] Switch :
    
|VLANs   |Noms   |ID Réseau    |Masques       |Ports      |
|--------|-------|-------------|--------------|-----------|
|VLAN 10 |RH     |192.168.10.0 |255.255.255.0 |fa0/1 - 8  |
|VLAN 20 |IT     |192.168.20.0 |255.255.255.0 |fa0/9 - 16 |
|VLAN 30 |PROD   |192.168.30.0 |255.255.255.0 |fa0/17 - 24|
    
- [x] Machines : 
    
|Noms|IP          |Masques      |Ports |
|----|------------|-------------|------|
|PC1 |192.168.10.1|255.255.255.0|Fa0/1 |
|PC2 |192.168.10.2|255.255.255.0|Fa0/2 |
|----|------------|-------------|------|
|PC3 |192.168.20.1|255.255.255.0|Fa0/9 |
|PC4 |192.168.20.2|255.255.255.0|Fa0/10|
|----|------------|-------------|------|
|PC5 |192.168.30.1|255.255.255.0|Fa0/17|
|PC6 |192.168.30.2|255.255.255.0|Fa0/18|
|----|------------|-------------|------|
|PC7 |192.168.10.5|255.255.255.0|Fa0/14|
    
## Topologie:

                 _________[Switch 1]________
                |             |             | 
                |             |             |
                |             |             |
            [PC1 - RH]    [PC3 - IT]    [PC5 - PROD]
            [PC2 - RH]    [PC4 - IT]    [PC6 - PROD]
            
            [VLAN 10 ]    [VLAN 20 ]    [VLAN 30   ]
            
## Étapes de Configuration:
> [!NOTE]
> Étape 1: Création des VLANs :

- [x] Entrer en mode Configuration globale:
```cisco
    Switch> enable
    Switch# configure terminal
    Switch (config)#
```
- [x] Passer en mode configuration de VLAN puis donner un nom à chaque VLAN:
```cisco
    Switch (config)# vlan 10
    Switch (config-vlan)# name RH
    Switch (config-vlan)# exit
    Switch (config)#
    
    Switch (config)# vlan 20
    Switch (config-vlan)# name IT
    Switch (config-vlan)# exit
    Switch (config)#
    
    Switch (config)# vlan 30
    Switch (config-vlan)# name PROD
    Switch (config-vlan)# end
    
    Switch#
```
- [x] Vérifier si les VLANS sont bien créés:
```cisco
    Switch# show vlan brief
    VLAN Name                             Status    Ports
    ---- -------------------------------- --------- -------------------------------
    1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                    Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                    Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                    Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                    Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                    Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                    Gig0/1, Gig0/2
    10   RH                               active    
    20   IT                               active    
    30   PROD                             active    
    
    Switch#
```
> [!WARNING]
> Résultat: Les trois VLANs sont bien créés et actives, mais tous les ports du switch reste dans le vlan par défaut (VLAN 1).
        
> [!NOTE]
> Étape 2: Configurer les ports d'accès par VLAN:

- [x] Entrer en mode configuration globale:
```cisco
    Switch#
    Switch# configure terminal
    Switch (config)#
```
- [x] Pour VLAN 10
    - Séléctionner une plage d'interface (ports) :
    ```cisco
        Switch (config)# interface range fa0/1 - 8
        Switch (config-if-range)#
    ```
    - Configurer le mode accès:
    ```cisco
        Switch (config-if-range)# switchport mode access
        Switch (config-if-range)# switchport access vlan 10
        Switch (config-if-range)# exit
        Switch (config)#
    ```
- [x] Pour VLAN 20
    ```cisco
        Switch (config)# interface range fa0/9 - 16
        Switch (config-if-range)# switchport mode access 
        Switch (config-if-range)# switchport access vlan 20
        Switch (config-if-range)# exit
        Switch (config)#
    ```
- [x] Pour VLAN 30
    ```cisco
        Switch (config)# interface range fa0/17 - 24
        Switch (config-if-range)# switchport mode access 
        Switch (config-if-range)# switchport access vlan 30
        Switch (config-if-range)# exit
        Switch (config)#
    ```
- [x] Vérification avec `show vlan brief` :
    ```cisco
        Switch#show vlan brief

        VLAN Name                             Status    Ports
        ---- -------------------------------- --------- -------------------------------
        1    default                          active    Gig0/1, Gig0/2
        10   RH                               active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                        Fa0/5, Fa0/6, Fa0/7, Fa0/8
        20   IT                               active    Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                        Fa0/13, Fa0/14, Fa0/15, Fa0/16
        30   PROD                             active    Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                        Fa0/21, Fa0/22, Fa0/23, Fa0/24
        Switch#
    ```
> [!WARNING]
> Résultat: les ports FastEthernet sont répartis dans les 03 VLANs créés

## Sauvegarder les configs 
    ```cisco
        Switch# write memory
    ```
                        
## Tests:
### Communication à l'intérieur du VLAN :
- Ping PC1 vers PC2 :
```
    C:\>ping 192.168.10.2

    Pinging 192.168.10.2 with 32 bytes of data:

    Reply from 192.168.10.2: bytes=32 time<1ms TTL=128
    Reply from 192.168.10.2: bytes=32 time<1ms TTL=128
    Reply from 192.168.10.2: bytes=32 time<1ms TTL=128
    Reply from 192.168.10.2: bytes=32 time<1ms TTL=128

    Ping statistics for 192.168.10.2:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 0ms, Maximum = 0ms, Average = 0ms

    C:\>
```
> [!WARNING]
> Résultat: PC1 envoie 04 paquets, et PC2 les réçoit sans perte (0% loss), ça prouve que les 02 PC communiquent sans problèmes.

- Ping PC3 vers PC4 :
```
    C:\>ping 192.168.20.2

    Pinging 192.168.20.2 with 32 bytes of data:

    Reply from 192.168.20.2: bytes=32 time<1ms TTL=128
    Reply from 192.168.20.2: bytes=32 time<1ms TTL=128
    Reply from 192.168.20.2: bytes=32 time<1ms TTL=128
    Reply from 192.168.20.2: bytes=32 time<1ms TTL=128

    Ping statistics for 192.168.20.2:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 0ms, Maximum = 0ms, Average = 0ms

    C:\>
```
> [!WARNING]
> Résultat: perte 0%, PC3 et PC4 se communiquent.

- Ping PC5 vers PC6 :
```
    C:\>ping 192.168.30.2

    Pinging 192.168.30.2 with 32 bytes of data:

    Reply from 192.168.30.2: bytes=32 time<1ms TTL=128
    Reply from 192.168.30.2: bytes=32 time<1ms TTL=128
    Reply from 192.168.30.2: bytes=32 time<1ms TTL=128
    Reply from 192.168.30.2: bytes=32 time<1ms TTL=128

    Ping statistics for 192.168.30.2:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 0ms, Maximum = 0ms, Average = 0ms

    C:\>
```
> [!WARNING] 
> Résultat: perte 0%, PC5 et PC6 se communiquent.
    
### Communication inter-vlan :
- Ping PC2 vers PC5 :
```
    C:\>ping 192.168.30.1

    Pinging 192.168.30.1 with 32 bytes of data:

    Request timed out.
    Request timed out.
    Request timed out.
    Request timed out.

    Ping statistics for 192.168.30.1:
        Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

    C:\>
```
> [!WARNING]
> Résultat: perte 100%, PC2 et PC5 ne peut pas se connecter.

- On branche une nouvelle machine (PC7) au port FastEthernet0/14
puis on lance un ping depuis PC7 vers PC1 :
```
    C:\>ping 192.168.10.1

    Pinging 192.168.10.1 with 32 bytes of data:

    Request timed out.
    Request timed out.
    Request timed out.
    Request timed out.

    Ping statistics for 192.168.10.1:
        Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

    C:\>
```
> [!WARNING]
> Résultat: perte 100%, PC7 et PC1 ne peut pa se communiquer
> parce que PC7 est branché dans un port d'accès pour VLAN 20,
> qui autorise seulement les traffics internes de ce VLAN et bloque
> tous les traffics vers les autres VLANs.
        
    
## Dépannages:

|Problèmes                    |Causes                             |Solutions                                  |
|-----------------------------|-----------------------------------|-------------------------------------------|
|Configuration perdues après redémarrage du switch|Oublie de la sauvegarde|Reconfigurer le switch puis sauvegarder les configs avec `write memory`|
|Ping PC5 vers PC6 échoue|Mauvaise configuration d'addresse IP dans PC6. J'ai écrit 192.168.30.3 au lieu de 192.168.30.2|Reconfigurer l'addresse IP de PC6|

## Commandes clés:

|Commandes|Utilisations|
|---------|------------|
|`vlan <num_vlan>`|Création de VLAN|
|`name <nom_vlan>`|Nommer/Renommer VLAN|
|`interface range <nom_interface0/num_début> - <num_fin>`|Sélectionner une plage d'interface (ex: `interface range fa0/1 - 10`)|
|`switchport mode access`|Activation du mode access|
|`switchport access vlan <num_vlan>`|Spécification du VLAN ayant l'accès|

> [!NOTE]
> Conclusion: Ce lab m'a permis de comprendre que: configurer un VLAN, c'est comme `construire des mûres`
> capable d'isoler les traffics des différents services. Mais ce sont juste des mûres, sans aucunne porte.
> Alors, au moment où un service a besoin d'accès à un autre, on doit passer au `routage inter-vlan`
> (router on a stick/switch L3).