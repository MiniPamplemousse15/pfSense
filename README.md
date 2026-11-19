# TP - Pfense - Mathis Pahu (1)

## A. Mise en place d’un package sous pfSense :

<aside>
📌

**Contexte**

Vous venez d’installer pfSense pour la société LSL et avez accès au serveur web situé dans la DMZ.

Le DSI de l’entreprise souhaite désormais pouvoir consulter, depuis un navigateur, un maximum d’informations sur le réseau LAN.

</aside>

### **Qu’est-ce que pfSense :**

![image.png](image.png)

**PfSense** est un logiciel open source basé sur FreeBSD qui permet de sécuriser et gérer un réseau informatique. Il agit comme un pare-feu et un routeur, en filtrant le trafic réseau, en créant des connexions VPN sécurisées et en contrôlant les accès à Internet grâce à une interface web simple d'administration.

### <u>I) Installation d’une VM pfsense sur Proxmox :</u>

![image.png](0ec00111-2f77-4254-96d9-755e0e0c3bce.png)

Après avoir téléversé l’image ISO, nous pouvons procéder à la création de la machine virtuelle, ce qui nous permettra ensuite d’installer **pfSense**.

Dès la création de la machine virtuelle terminée, nous devons ajouter trois interfaces réseau :

- **vmbr0** → **WAN**
- **lanMP** → **LAN**
- **dmzMP** → **DMZ**

![image.png](image%201.png)

Cette configuration permet à pfSense de gérer les communications entre le réseau externe (WAN), le réseau interne (LAN) et la zone démilitarisée (DMZ).

Après avoir installé et démarré la machine virtuelle, nous procédons aux étapes d’installation et de configuration :

![00.1 installtion de l'interface wan vmbr0.png](00.1_installtion_de_linterface_wan_vmbr0.png)

![00.2 installation de l'interace lan vmbr lanMP.png](00.2_installation_de_linterace_lan_vmbr_lanMP.png)

***Nous configurons l’interface WAN en l’associant à l’interface réseau vmbr0.***

***Puis nous configurons l’interface LAN de notre pfSense en l’associant à l’interface réseau  lanMP.***

![00.3 true - false et configuration de l'ip interne.png](00.3_true_-_false_et_configuration_de_lip_interne.png)

![00.4 install.png](00.4_install.png)

***Nous désactivons le service DHCP (True → False) afin d’éviter l’attribution automatique des adresses IP par pfSense. Nous configurons ensuite une adresse IP statique 192.168.23.148 pour l’interface WAN.***

Enfin, nous validons l’ensemble de nos configurations, puis nous lançons l’installation définitive de pfSense. Une fois l’installation terminée, nous arrivons à l’interface suivante :

![00.6 fin de l'install.png](00.6_fin_de_linstall.png)

**Notre WAN est en 192.168.20.177***

### <u>II) Accès à l’interface Web de pfSense depuis un client Debian</u>

**Tout d’abord, nous affectons l’interface réseau lanMP à notre VM Debian**

![01.0 carte réseau modifier.png](01.0_carte_rseau_modifier.png)

Depuis notre client Debian, nous configurons une adresse IP statique 192.168.23.149 et définissons comme passerelle par défaut l’adresse IP de notre pfSense, 192.168.23.148, afin de permettre la communication entre les deux machines.

![01.2 configuration ip de la debian graph.png](6867d91e-7625-4f76-baa1-0d6c00425eb2.png)

![01.4 ping vers pfsense fonctionne.png](01.4_ping_vers_pfsense_fonctionne.png)

Le test de connectivité effectué à l’aide de la commande `ping` confirme que notre client Debian et pfSense communiquent correctement entre eux.

<u>***Voici la topologie actuelle :***</u>

![01.6 topologie actuel.png](bcb4bd51-a603-42da-b2bb-9a8d6a42953c.png)

Depuis un navigateur Web sur notre client Debian, nous accédons à l’interface d’administration de pfSense en saisissant son adresse IP : 192.168.23.148.

![01.7 ouvrir depuis la graph.png](01.7_ouvrir_depuis_la_graph.png)

Nous nous connectons en entrant l’id → admin et le mot de passe → pfsense :

![02.0 connexion au serveur.png](02.0_connexion_au_serveur.png)

Pour affecter l’interface réseau dédiée à la DMZ, nous suivons le chemin suivant dans l’interface Web de pfSense : Accueil → Interfaces → Assignments → Add (+).

![02.2 dans les interfaces.png](02.2_dans_les_interfaces.png)

Nous sélectionnons ensuite l’interface OPT1 (vtnet1), puis nous la renommons dmzMP afin d’identifier clairement l’interface associée à notre zone démilitarisée (DMZ).

![02.3 apres Add OPT1 et crée.png](02.3_apres_Add_OPT1_et_cre.png)

On affecte l’ip de la DMZ : 

![02.6 IPV4.png](02.6_IPV4.png)

Puis nous retrouvons bien nos 3 interfaces : 

![02.7 tous est bon.png](02.7_tous_est_bon.png)

Processus pour créer une règle : 

![02.8.1 ajout d'une regle.png](02.8.1_ajout_dune_regle.png)

![02.8.2 ajout ip.png](02.8.2_ajout_ip.png)

Nous configurons une règle NAT dans Firewall → NAT → Port Forward afin de rediriger les requêtes HTTP entrantes sur l’interface WAN de pfSense vers le serveur interne 192.168.148.1, via le protocole TCP et le port 80. 

**Cette règle permet de rendre le serveur Web accessible depuis le réseau externe, sans exposer directement l’ensemble du réseau interne.**

### <u>III) Test d’accès à l’interface Web d’un serveur LAMP hébergé dans la DMZ</u>

![03.0 carte réseau du LAMP.png](03.0_carte_rseau_du_LAMP.png)

Nous configurons notre serveur LAMP pour qu’il soit sur la même interface réseau que notre DMZ → dmzMP.

Nous configurons notre serveur LAMP avec l’adresse IP statique 192.168.148.10, afin qu’il soit accessible sur le réseau de la DMZ et puisse recevoir les requêtes HTTP redirigées par pfSense.

Nous effectuons un test d’accès depuis le réseau WAN, à l’aide de l’adresse IP 192.168.20.177. Le test est concluant : l’interface Web de notre serveur LAMP, hébergé dans la DMZ, est accessible grâce à la règle de redirection NAT configurée sur pfSense.

![06.1 Test LAMP.png](06.1_Test_LAMP.png)

<u>***Topologie finale de la situation A :***</u>

![Diagramme sans nom.drawio (2).png](Diagramme_sans_nom.drawio_(2).png)

## B. Mise en place d'un bureau distant sous pfSense :

<aside>
📌

**Contexte**

Le DSI souhaite maintenant installer un serveur Windows dans la DMZ
accessible en bureau à distance via le protocole RDP sur le port par défaut,et ce depuis internet .

Par mesure de sécurité vous utiliserez un port différent de votre choix
associé à l’adresse WAN du PFSense avec redirection du flux vers
l’adresse du serveur Windows associé au port par défaut .

</aside>

Nous configurons une règle de redirection NAT sur l’interface WAN de pfSense afin de rediriger le trafic TCP entrant sur le port **14823** vers notre serveur interne 192.168.148.11, sur le port RDP (3389). Cette règle permet d’accéder à distance au serveur via le protocole Bureau à distance (RDP)

![04.0 regle pour le windows server 2019.png](04.0_regle_pour_le_windows_server_2019.png)

![04.1 suite de la regle.png](04.1_suite_de_la_regle.png)

<u>**Récapitulatif des règles configurées :**</u>

![05.1 regles globales.png](05.1_regles_globales.png)

- Règle Block bogon networks : bloque les réseaux réservés ou non attribués sur Internet.
- Règle DMZ subnets : autorise le trafic provenant des sous-réseaux de la DMZ.
- Règle DMZ subnets : autorise également le trafic provenant des sous-réseaux de la DMZ selon les paramètres définis.
- Règle DMZ subnets : permet le trafic à destination de l’adresse IP `192.168.148.1`.
- Règle NAT HTTP : redirige les requêtes entrantes vers le serveur LAMP `192.168.148.10` sur le port `80` (HTTP).
- Règle NAT RDP : redirige les connexions entrantes sur le port externe `14823` vers le serveur `192.168.148.11` sur le port `3389` (RDP).

Nous configurons une adresse IP statique 192.168.148.11 sur notre Windows Server 2019, avec comme passerelle l’adresse IP de l’interface DMZ de pfSense. Depuis notre poste connecté au réseau WAN, nous établissons ensuite une connexion en Bureau à distance (RDP) via le port 14823. Le test est concluant : nous parvenons à accéder à notre serveur Windows à distance.

![06.2 Test bureau a distn.png](a847605c-74bb-4389-89e8-ddc68eded10b.png)

![06.3 test distance serveur.png](ca963e09-a2d7-4a33-bcff-948a587d0b43.png)