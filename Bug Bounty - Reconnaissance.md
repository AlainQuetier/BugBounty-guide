# Website reconnaissance

## Manually  

étapes:  
- Observer le site comme un utilisateur lambda
- Cliquer sur tout les liens pour voir ou l'on va et comment fonctionne le site
- Utiliser toutes les fonctionnalités possible
- Si c'est possible, créer des comptes à différents niveaux de privilèges


Ceci nous permettra de découvrir notre surface d'attaque, la façon dont l'application web est perçue par chaque rôle utilisateur.  

## Dorkings 

Liste courte de dorks pour Google:  

| Dork | Fonctionnalité |
| :--: | :-- |
| **site**  | Permet de cibler un site précis pour une reconnaissance d'info |
| **inurl** | Permet de trouver des infos précise qui sont dans une url |
| **intitle** | Permet de trouver un élément qui est dans le title d'une page |
| **link** | Permet de trouver des liens qui sont dans les pages |
| **filetype** | Permet de cibler un type de fichier à trouver |
| **\*** | Le wildcard est un caractère de remplacement pour un caractère ou une chaine complète |
| **" "** | Force à chercher une correspondance exacte d'un terme |
| **\|** | Le pipe est un OR qui permet de chercher un terme OU un autre |
| **-** | Le minus permet d'exclure un terme d'une recherche |
| **ext** | Permet de chercher après une extension précise |

*Note: Penser à utiliser aussi Bing et Yandex pour approfondir ses recherches*  
*Note2: Google et Yandex donnent un captcha, bing n'en a pas peu importe le nombre de requêtes*

Pour voir des dorks supplémentaires, on peut voir ici :  [Google Hacking DB]('https://www.exploit-db.com/google-hacking-database')

## Découverte de scopes  

Attention, lors de la découverte de scope, bien vérifier les domaines/produits/services/ip attaquable pour ne pas être hors du contexte.  

- Utiliser **Whois** pour obtenir des infos avec un nom de domaine
- Utiliser un [Reverse Whois]('https://viewdns.info/reversewhois/') pour des infos a partir d'un mail etc.
- Utiliser **NSlookup** pour obtenir l'IP du DNS et effectuer une recherche plus profonde
- Utiliser **crt.sh**, **rapiddns**, **censys**, **cert spotter** pour trouver les sous domaine existant via le certificat
- Enumérer les subdomains à l'aide de **FFUF**, **AMASS**, **Sublist3r**, **SubBrute**, **AltDNS**...
- Découvrir les services à l'aide de **NMAP** ou **MASSCAN** (recherche ACTIVE !!!!)
- Découvrir les services à l'aide de **Shodan**, **Censys**, **Project Sonar** (recherche PASSIVE !!!!)
- Fuzz les directorys avec **dirb**, **gobuster**, **FFUF**...
- Osint sur Pastebin, profile des employé pour les technos, et github
- Découverte des 3rd-Party-Host (**AWS**, **Azure**, Etc..) avec cet outil : [Bucket Discover]("https://buckets.grayhatwarfare.com/")

### Manipulations speciale AWS

Ci dessous, plusieurs informations sur les failles potentielles liées à des buckets AWS.

>Installer AWS CLI avec la commande *"pip install awscli"*
>Configurer le CLI à l'aide de cette documentation [AWS CLI CONFIGURATION]("https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html")

Une fois la configuration faite, on peut tenter d'accéder aux buckets et de faire un listing

> aws s3 ls s3://*BUCKET_NAME*/

Si ça fonctionne, on peut tenter de lire le contenu en copiant du bucket vers notre machine

> aws s3 cp s3://*bucket_name*/*filename*/*lien*/*vers*/*repertoire*/*local*

**ATTENTION: Un bucket s3 exposé est généralement considéré comme une vulnérabilité.**

On peut également tenter d'upload un fichier et de le supprimer, ce qui nous permettra de voir les intérations possible sur le bucket.

*Upload de notre machine vers le bucket*  
>aws s3 cp *FichierLocal* s3://*BUCKET_NAME*

*Suppression de notre fichier dans le bucket*
>aws s3 rm s3://*BUCKET_NAME*/*FichierLocal*

Ces deux commandes permettent de montrer le droit d'écriture et suppression sur le buckets sans faire de dégats. 
