# Open Redirect

## Qu'est-ce qu'un Open Redirect

Les sites utilisent souvent des paramètres http ou url pour faire une redirection dans un service (exemple, je vais sur une page, et ça me rebascule sur la page de login). 

L'**Open Redirect** consiste à manipuler la valeur de ces paramètres et à rediriger sur une page autre que celle attendue.

## Trouver une Open Redirect

Un premier exemple est de regarder après les paramètres de redirection.  
Exemple:  

>https://exemple.com/login?redirect=https://exemple.com/dashboard  
>https://exemple.com/login?next=/dashboard  
>https://exemple.com/dashboard?url=https://exemple.com/login  

**ATTENTION**  
Les noms de paramètres de redirection d'URL peuvent aussi ne pas être implicite.

Ici je vais mettre une liste non exaustive qui sera tenue a jour pour chaque paramètre rencontré et qui corresponds à une redirection:  

- ?redirect=  
- ?redir=  
- ?next=  
- ?u=  
- ?n=  
- ?RelayState=  
- ?forward=  

S'il n'y a pas de paramètre d'URL, alors il faut prêter attention aux **Status code 3xx** tel que les **301** et **302** qui redirigent automatiquement les pages, et qui sont des redirection "**potentiellement**" vulnérable a un open redirect par "**Referer**''.

---

On peut également utiliser des Google dorks pour trouver des pages contenant des redirections.

>inurl:"%3Dhttp" site:"exemple.com"  

Ce qui nous donnera des résultats tels que :  
>http://exemple.com/buy?redir=http://exemple.com/log+in  
>https://exemple.com/log?n=https://exemple.com/dashboard  

Un autre exemple:  

>inurl:"%3D%2F" site:"exemple.com"

Qui nous donnes des résultats tels que :   
>http://exemple.com/log?n=/dashboard

Et finalement, on peut aussi chercher après le nom d'un paramètre que l'on veut:  

>inurl:"redirect" site:"exemple.com"  
>inurl:"red" site:"exemple.com"  
>inurl:"next" site:"exemple.com"    

Qui nous donnes des résultats tels que :  
> http://exemple.com/log?redirect=/dashboard  
> https://exemple.com/buy?next=https://exemple.com/checkout  
> https://exemple.com/pay?red=cart  

---   
## Bypass de protection Open Redirect  

### Utiliser l'autocorrection du navigateur

Les navigateurs modernes ont tendance à corriger la syntaxe des composants url.   
Par exemple, toutes les urls ci dessous redirigent vers "http://evil.com"   

>https:evil.com  
>https;evil.com  
>https:\\/\\/evil.com  
>https:/\/\evil.com  
>http:\\\\evil.com  

Si le validateur d'url ne prends pas le "\\" en tant que séparateur, alors on peut tenter une open redirect de ce genre :  

>http://evil.com\good.com  

Qui va interpréter la première partie comme étant le bon endroit, et la seconde partie comme un fragment d'url.  

### Exploiter les erreurs de logique du validateur d'url

En général les validateurs d'url acceptent l'url de redirection quand :   
- L'url commence par le nom de domaine  
- L'url contient le nom de domaine  
- L'url finit par le nom de domaine  

Ce qui nous laisses des options simple pour bypass ces filtres.   
Je donnes un exemple pour les trois validations au dessus:  

Si le validateur attends que l'on ai le début de l'url, ou fin d'url alors on peut créer un sous domaine du nom de notre victime ou bien créer un dossier avec le nom de domaine de la victime.  
> https://good.com/log?redirect=https://good.com.evil.com   
> https://good.com/log?next=http://evil.com/good.com  

Parfois, le validateur attends que l'ont ai le début ET la fin du nom de domaine, que l'on peut bypass de cette manière:  

>https://good.com/buy?r=http://good.com.evil.com/good.com  

Si on veut, on peut essayer de jouer avec l'url en ajoutant un "@" qui ferait que le premier fragment d'url serait considéré comme un username, et le reste comme étant le NDD a utiliser:  

>https://good.com/log?url=https://good.com@evil.com/good.com  

Il y a beaucoup de cas possible utilisable car les développeurs ne pensent pas forcément à tout les cas possible.  

### Utiliser les data urls  

On peut bypass aussi la vérification en utilisant les schéma "data:" dans l'url et en prenant soin d'encoder en base64 notre url malicieuse  :  

>https://good.com/log?r=data:text/html;base64,aHR0cHM6Ly9nb29kLmNvbUBldmlsLmNvbS9nb29kLmNvbQ==

Ce qui donnes une fois passé par le validateur:  

>https://good.com/log?r=https://good.com.evil.com/good.com  

Si la cible est vulnérable à une xss et a l'injection du schema "data" dans l'url, alors on peut utiliser un payload comme celui ci:  

>https://good.com/log?r=data:text/html;base64,PHNjcmlwdD5sb2NhdGlvbj0iaHR0cDovL2V2aWwuY29tIjwvc2NyaXB0Pg==  

Qui va être traduit comme ceci :  

>https://good.com/log?r=<script>location="http://evil.com"</script>  

qui redirigera notre victime sur notre site malveillant, et exécutant le script que l'on veut.  

### utiliser le double/triple encodage  

Une autre méthode pour bypass la protection est l'encodage url en double voir triple.  

Exemple, lorsque nous appelons une url, pour qu'elle soit bien prise en charge par le navigateur, elle est encodée en hexadécimal précédé par le caractère "%".  

Si nous faisons un payload en encodage simple, le "/" serait représenté par l'hexa "%2f".  

>https://good.com%2f@evil.com  

On peut également double encoder et triple encoder ce qui donnes :  

>https://good.com%252f@evil.com
>https://good.com%25252f@evil.com  

Ce qui pourrait déclencher une erreur de logique et donc interpréter notre première partie comme un fragment d'username et la seconde partie comme notre site legit sur lequel il faut rediriger.

L'inverse est également possible, car si le décodage se fait, on pourrait tenter ceci:  

>https://evil.com%252f@good.com  

Il considérera good.com comme étant le NDD, mais sera interprété par le navigateur comme étant un répertoire "/good.com" et nous redirigera quand même sur evil.com.

### Utiliser des caractères non ASCII  

Une autre manière est l'utilisation de caractère non ASCII, exemple:  

> https://evil.com%ff.good.com  

Le navigateur peut réagir de plusieurs manières différentes, l'une d'elle est de remplacer le caractère non ascii par un "?" ce qui donnera ce résultat:  

>https://evil.com?.good.com  

La conséquence est que l'url sera notre url malicieuse, et que le reste est considéré comme une partie de requête qui sera ignorée et donc qui nous redirigera sur notre page.  

Une autre méthode aussi est que les navigateurs peuvent avoir envie de trouver un caractère auquel il ressemble le plus.  

Par exemple, si nous utilisons le caractère "%E2%95%B1" qui est "╱", le navigateur va vouloir l'interprêter comme étant un "plus proche du caractère /" et donc l'interprêter comme un slash.  

>https://evil.com%E2%95%B1.good.com  
>https://evil.com╱.good.com  
>https://evil.com/.good.com  

Pour le genre de caractères en "look alike", on peut regarder les caractères unicodes et plus particulièrement le **cyrillique** qui contient pas mal de caractère proche de l'ascii.  

### Combiner les techniques  

Pour bypass les filtres les plus récalcitrants, on peut combiner les techniques de bypass.  
Par exemple, celui ci-dessous permet de valider que :  

- l'url débute par le bon domaine
- l'url fini par le bon domaine
- l'url contient le bon domaine

Mais également d'embrouiller le navigateur en faisant penser que le premier fragment d'url est la portion username en laissant donc l'url malicieuse être la page vers laquelle l'utilisateur est redirigé.  

>https://good.com%25252f@evil.com%E2%95%B1good.com

Ce qui serait interprété au final par:  

>https://good.com/@evil.com/good.com  

Une autre méthode qui peut être efficace pourrait être celle ci:  

>https://good.com/?next=http://evil.com/

En effet, parfois la double redirection n'est pas prise en compte et bypassera le filtrage.  

### Quels sont les vecteur d'escalade possible  

Account Takeover : L'attaquant envoie l'url "légit" à la victime. La victime entre ses coordonnées sur le vrai site, mais est redirigé a la validation de ses informations vers la copie du site sur l'url de l'attaquant en précisant que les creds sont faux, obligeant la personne à ressaisir ses infos.

SSRF : L'utilisation d'un open redirect peut maximiser l'impact d'une SSRF. Si un site utilise une liste blanche en prévention de SSRF et n'autorise que quelques pages a exécuter des requêtes, l'open redirect permettrais l'utilisation de n'importe quelle page autoriser pour rediriger la requête n'importe où.
