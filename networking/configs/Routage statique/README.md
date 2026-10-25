# Configuration des routes statiques dans un routeur CISCO

## Objectif:
#### - Configurer des routes statiques pour faire communiquer deux réseaux différents

## Environnement:
- [x] Simulateur : Cisco Packet Tracer
- [x] Machines :

|Nom d'hôtes|Addresse IP|Masque de sous-réseaux|Passerelle   |
|-----------|-----------|----------------------|-------------|
|PC1        |192.168.1.1|255.255.255.0         |192.168.1.254|
|PC2        |192.168.2.1|255.255.255.0         |192.168.2.254|
    
- [x] Routeurs (Cisco Catalyst 2911):
    
|Nom d'hôtes|Interfaces        |Addresse IP   |Masque de sous-réseaux|
|-----------|------------------|--------------|----------------------|
|Router 1   |GigabitEthernet0/0|192.168.1.254 |255.255.255.0         |
|           |GigabitEthernet0/1|10.0.0.1      |255.255.255.252       |
|-----------|------------------|--------------|----------------------|
|Router 2   |GigabitEthernet0/0|192.168.2.254 |255.255.255.0         |
|           |GigabitEthernet0/1|10.0.0.2      |255.255.255.252       |

## Topologie:
        
    [PC1]--------------[Router 1]-----------[Router 2]--------------[PC2]

## Étapes de Configuration:
> [!NOTE] 
> Étape 1: Assignation d'addresse IP et démarrage d'interface:
- [x] Routeur 1 :
    - Entrez en mode Configuration globale:
    ```cisco
        Router> enable
        Router# configure terminal
        Router (config)#
    ```
    - Séléctionner l'interface GigabitEthernet0/0:
    ```cisco
        Router (config)# interface GigabitEthernet0/0
        Router (config-if)#
    ```
    - Assigner l'addresse IP puis démarrer l'interface:
    ```cisco
        Router (config-if)# ip address 192.168.1.254 255.255.255.0
        Router (config-if)# no shutdown
        Router (config-if)# exit
        Router (config)#
    ```
    - Séléctionner l'interface GigabitEthernet0/1:
    ```cisco
        Router (config)# interface GigabitEthernet0/1
        Router (config-if)# ip address 10.0.0.1 255.255.255.252
        Router (config-if)# no shutdown
        Router (config-if)# exit
        Router (config)#
    ```
- [x] Routeur 2 :
    - On fait la même principe que la configuration du Routeur 1:
    ```cisco
        Router> enable
        Router# configure terminal
        Router (config)# interface GigabitEthernet0/0
        Router (config-if)# ip address 192.168.2.254 255.255.255.0
        Router (config-if)# no shutdown
        Router (config-if)# exit
        
        Router (config)# interface GigabitEthernet0/1
        Router (config-if)# ip address 10.0.0.2 255.255.255.252
        Router (config-if)# no shutdown
        Router (config-if)# exit
        Router (config)#
    ```
> [!NOTE]
> Étape 2: Configuration des routes statiques:
- [x] Routeur 1 :
    - Configurer une route statique pour atteindre le réseau 192.168.2.0 via Routeur 1:
    ```cisco
        Router (config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
        Router (config)# end
        Router#
    ```
    - Vérification avec `show ip route static` (afficher les routes connues par le routeur):
    ```cisco
        Router# show ip route static
        S    192.168.2.0/24 [1/0] via 10.0.0.2

        Router#
    ```
    
- [x] Routeur 2 :
    - Configurer une route statique pour atteindre le réseau 192.168.1.0 via Routeur 1:
    ```cisco
        Router (config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
        Router (config)# end
        Router#
    ```
    - Vérification avec `show ip route static`:
    ```cisco
        Router# show ip route static
        S    192.168.1.0/24 [1/0] via 10.0.0.1

        Router#
    ```
> [!NOTE]
> Étape 3: Enregistrement des configurations actuelles dans la mémoire NVRAM :
    ```cisco
        Router# copy running-config stratup-config
    ```
        OU
    ```cisco
        Router# write memory
    ```
            
## Tests:
- [ ] Sans route statique:
    - Ping PC1 vers PC2:
    ```
        C:\>ping 192.168.2.1

        Pinging 192.168.2.1 with 32 bytes of data:

        Reply from 192.168.1.254: Destination host unreachable.
        Reply from 192.168.1.254: Destination host unreachable.
        Reply from 192.168.1.254: Destination host unreachable.
        Reply from 192.168.1.254: Destination host unreachable.

        Ping statistics for 192.168.2.1:
            Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

        C:\>
    ```
> [!WARNING]
> Résultat:
> on a envoyé 04 paquets, mais ils sont tous perdus (100% loss),
> cela montre que la que PC1 et PC2 n'arrive pas à se communiquer sans les routes statiques
> configurées dans les routeurs.

- [ ] Avec route statique:
    - Ping PC1 vers PC2:
    ```
        C:\>ping 192.168.2.1

        Pinging 192.168.2.1 with 32 bytes of data:

        Reply from 192.168.2.1: bytes=32 time<1ms TTL=254
        Reply from 192.168.2.1: bytes=32 time<1ms TTL=254
        Reply from 192.168.2.1: bytes=32 time<1ms TTL=254
        Reply from 192.168.2.1: bytes=32 time<1ms TTL=254

        Ping statistics for 192.168.2.1:
            Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
        Approximate round trip times in milli-seconds:
            Minimum = 0ms, Maximum = 0ms, Average = 0ms

        C:\>
    ```
> [!WARNING]   
> Résultat:
> on a envoyé 04 paquets, et ils sont tous réçus par PC2 (0% loss),
> cela montre que les 02 PC peuvent se communiquer et échanger des données à partir
> des routeurs: Routeur 1 et Routeur 2.

## Dépannage:
|Problèmes                    |Causes                                                    |Solutions                            |
|-----------------------------|----------------------------------------------------------|-------------------------------------|
|Ping échoue encore même avec |Conflit d'addresse IP entre Routeur 2 (GigabitEthernet0/0)|Changement de l'addresse IP          |
|routage statique configuré   |et PC2                                                    |du routeur                           |

## Commandes clés:
|Commandes|Utilisations|
|---------|------------|
|`interface <nom_interface>`|Séléctionner une interface réseau|
|`ip address <ip> <mask>`|Assigner une addresse IP à l'interface séléctionner|
|`no shutdown`|Démarrer une interface|
|`ip route <réseau_destination> <mask> <ip_next_hop>`|Création de route statique|
|`show ip route` ou `show ip route static`|Afficher la table de routage du routeur|

> [!CAUTION]
> Pour supprimer une route statique, il suffit d'ajouter `no` devant la Commande de création `no ip route <réseau_destination> <mask> <ip_next_hop>`
    
> [!NOTE]
> Conclusion : 
> Ce lab m'a permis de comprendre que les routeurs `transfèrent`
> les paquets reçus de l'une de ses interfaces vers l'autre puis à son voisin
> selon les routes statiques configurées par l'administrateur réseau.
> Et au moment où ils ne connaissent pas les routes vers la destination du paquet,
> ils les `rejetent`.
