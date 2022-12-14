# *Brief 5*
### Raja 


## *Chapitre 1 - Créer un sous-domaine*

* **Nom du domaine**

<img width="960" alt="nom domain" src="https://user-images.githubusercontent.com/108053084/193400645-17e1a9f8-dd31-474e-b04f-4bc79a0d2004.PNG">

* **Simplon-raja.space**

<img width="926" alt="simplonrajaspace" src="https://user-images.githubusercontent.com/108053084/193400723-48f1b41c-2f2e-4bd9-9559-23942a8c6854.PNG">

* **Enregistrement DNS**

<img width="949" alt="enregistre le DNS" src="https://user-images.githubusercontent.com/108053084/193400820-93c2b6fb-cacb-42d5-96e1-ebd20c480ccc.PNG">

* **Enregistrement DNS2**

<img width="895" alt="enregistre le DNS2" src="https://user-images.githubusercontent.com/108053084/193400983-648dac32-abd8-4263-b061-a14dead5ff7d.PNG">



*   **Type** -> *A*    
*   **Nom** ->*Vote.simplon-raja.space*
*   **Adresse IP V4** -> *IP gateway* (pour déterminer l'adresse ip de la gateway il faut lancer le script de test Alfred sur terraform, puis on va sur Azure et on récupère l'adresse IP de la gateway et non de la VM, car la VM est liée à la gateway). 


## *Chapitre 2 - Créer un certificat*

        * sudo apt-get update 
        * sudo apt-get upgrade 
        * sudo apt install software-properties-common -y
        * sudo add-apt-repository ppa:deadsnakes/ppa -y
        * sudo apt install python3.10
        * sudo apt-get update
        * sudo apt-get upgrade
        * sudo su (root)
        * pip3 install certbot-plugin-gandi


*2.2 Après avoir téléchargé python3.10 sur Ubuntu 20.04, j'ai créé un fichier main.py et un autre gandi_api.py.* 


1. Obtain a Gandi API token (see Gandi LiveDNS API)
* **Paramètres**

<img width="937" alt="paramètres" src="https://user-images.githubusercontent.com/108053084/193401753-d1ed8aab-5690-44af-b66a-fc5df77b8ea3.PNG">

* **Compte et sécurité**

<img width="805" alt="compte et sécurité" src="https://user-images.githubusercontent.com/108053084/193401835-18a407b8-d80b-4bfa-ac4c-f814b8d4094d.PNG">

* **Sécurité** -> Regénérer la clef d'Api. (Après avoir cliqué sur "Regénérer la clef d'Api", nous allons obtenir un mot de passe qu'il faudra copier coller plus tard sur un fichier)

<img width="737" alt="sécurité" src="https://user-images.githubusercontent.com/108053084/193401945-861a6e5a-f8b2-41d6-8e41-052f49ba4fd2.PNG">

*2. Install the plugin using pip install certbot-plugin-gandi*

*3. Créer un fichier gandi.ini*
    
        * # live dns v5 api key
        dns_gandi_api_key= (mettre la clef d'Api que on avez genéré dans le premier point)
        
        * chmod 600 gandi.ini
    
    
*4. Exécutez certbot et indiquez-lui d'utiliser le plugin pour l'authentification et d'utiliser le fichier de configuration créé précédemment :*
             
            *  sudo su
      
            *  certbot certonly --authenticator dns-gandi --dns-gandi-credentials gandi.ini -d vote.simplon-raja.space
        
        
* Quand on a ce type d'erreur, on fait la commande: 
     
         *  cat /var/log/letsencrypt/letsencrypt.log  
         
- [nous permettra de voir où est l'erreur, dans ce cas-ci l'erreur était due au fait que je n'avais pas correctement pris la clef d'Api, j'ai par erreur copier et coller mon mot de passe au lieu de prendre la clef d'Api.]


## *Chapitre 3 - Charger le certificat dans Azure Application Gateway*

``` consol
    
        * certbot certonly --authenticator dns-gandi --dns-gandi-credentials gandi.ini -d brief6.simplon-raja.space
        * * cd /etc/letsencrypt/live/vote.simplon-raja.space
        * ls
        * cat cert.pem privkey.pem > certificat.pem
        * ls 
        * mv certificat.pem /home
        * cd /home
        * ls
        * mv certificat.pem /home/raja
        * cd raja
        * ls 
        * openssl pkcs12 -export -out certificat.pfx -in certificat.pem
        
``` 

* *4.a. Activer le HTTPS sur l’Application Gateway sur le port 443*


* **Ecouteurs**

<img width="943" alt="écouteurs" src="https://user-images.githubusercontent.com/108053084/193404376-99e59897-f0a3-4c6e-90b8-e8c49acd9e4d.PNG">


* **Ajouter un écouteur**
* **Nom de l'écouteur** -> *listener*
* **Adresse ip du front-end** -> *Publique*
* **Protocole** -> *HTTPS*
* **Port** -> *443* 
* **Sélectionner un certificat** -> *Créer*
* **Paramètres HTTPS** -> Télécharger un certificat 
     ->  Nom du certificat: vote
     ->  Fichier de certificat PFX -> 
     
     <img width="464" alt="wsl" src="https://user-images.githubusercontent.com/108053084/193405135-e515789b-78a2-4170-85f0-26c5ed9d347c.PNG">

* **Home -> Raja -> certificat pfx -> ajouté**
* **Mot de passe** -> "*****"
* **Paramètres supplémentaires** :
-> *Type d'écouteur* -> *de base*
-> *URL de la page d'erreurs*  -> *no*
            
     <img width="461" alt="wsl2" src="https://user-images.githubusercontent.com/108053084/193405188-559a2574-7cd1-4f9e-abcd-aca191ce183b.PNG">
     
* Ajouter les règles -> *Règles d'acheminement*

     <img width="942" alt="règles" src="https://user-images.githubusercontent.com/108053084/193407224-c28cf981-a41a-43b3-9bc9-f83056a5fe36.PNG">
     
* **rule-1** -> 
**Nom de la règle** : *rule-1*
**Priorité** : 100

**Cibles de back-en** ->
**Type de cible** -> Redirection 
**Type de rediction** -> *Permanent*
**Cible de rediction** -> *HTTPS*
**Inclure la chaîne de requête** -> *OUI*
**Inclure le chemin** -> *OUI*

* **rule-2** -> 
**Nom de la règle** -> *rule-2*
**Priorité** : 103
**Ecouteurs** -> *HTTPS*
**Cibles de back-end** ->
**Type de cible** -> *Pool principal*
**Cible de back-end** -> *backend-pool* 
**Paramètres de back-end** -> *https-setting*


## *Chapitre 4 - Charger le certificat dans Azure KeyVault*

*1. Créer un KeyVault sur Azure en utilisant la CLI*

``` consol

- Power shell:
        *    az login 
        *    az account show
        *    az account list --all --output table
        *    az group create --name "testraja" -l "westus"
        *    az keyvault create --name "keyvaultrajach" --resource-group "testraja" --location "westus"
        
```

*2. Importer le fichier contenant le certificat et la clé dans le KeyVault en utilisant la CLI.*

      *  az keyvault certificate import --vault-name keyvaultrajach -n certif -f C:\Users\rajac\Desktop\certificat.pfx --password 12345

*3 Vérifier que le certificat est dans le KeyVault en utilisant la CLI : az keyvault certificate show -v <vault-name> -n <certificate-name>*
    
    *  az keyvault certificate show --vault-name keyvaultrajach  -n certif
