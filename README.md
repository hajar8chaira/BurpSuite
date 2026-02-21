# Lab3 >> BurpSuite


<p align="center"> <img src="images/z1.png" width="300"> </p>
<p align="center"> <img src="images/z2.png" width="300"> </p>
<p align="center"> <img src="images/z3.png" width="300"> </p>
<p align="center"> <img src="images/z4.png" width="300"> </p>
<p align="center"> <img src="images/z5.png" width="300"> </p>
<p align="center"> <img src="images/z6.png" width="300"> </p>
<p align="center"> <img src="images/z7.png" width="300"> </p>
<p align="center"> <img src="images/z8.png" width="300"> </p>
<p align="center"> <img src="images/z9.png" width="300"> </p>
<p align="center"> <img src="images/z12.png" width="300"> </p>

## Analyse du trafic HTTP dans Burp Suite

### Proxy → HTTP history

Une ligne correspond à une requête HTTP.

### Colonnes importantes

1. **Host** : domaine contacté (ex: httpbin.org)  
2. **Method** : GET, POST  
3. **URL** : chemin de la ressource (/ , /get, etc.)  
4. **Status code** : 200, 404, 500  
5. **Length** : taille de la réponse  
6. **MIME type** : HTML, JSON  
7. **Time** : heure de la requête  

### Observations

- Requêtes GET vers httpbin.org  
- Statut 200 OK  
- Réponses en HTML et JSON  

Cela signifie que le proxy fonctionne correctement et intercepte bien le trafic.
<p align="center"> <img src="images/z10.png" width="300"> </p>
<p align="center"> <img src="images/z11.png" width="300"> </p>

# Analyse d’une requête HTTP interceptée (Instagram AJAX)

- Méthode : POST
- Endpoint : /ajax/qm/
- Protocole : HTTP/2
- Cible : https://www.instagram.com
- Environnement : Chrome Android (émulateur)

---

## 1. Request Attributes

Analyse :

- Requête de type POST → envoi de données au serveur.
- Endpoint interne AJAX.
- Probable mécanisme de télémétrie / tracking.

---

## 2. Request Query Parameters


Analyse :

- __a et __comet_req : paramètres internes du framework Meta.
- __user=0 : session non authentifiée ou anonyme.
- jazoest : token interne lié à la protection CSRF.

Aucune donnée sensible identifiable dans les query parameters.

---

## 3. Request Body Parameters

Analyse :

- event_id : identifiant unique d’événement (tracking).
- marker_page_time : mesure de performance.
- script_path : chemin du script exécuté.
- lsd : token interne de validation.

Ces paramètres indiquent une requête de télémétrie et non une action sensible.

---

## 4. Request Cookies

Analyse :

- csrftoken : protection CSRF.
- datr : identifiant navigateur Meta.
- ig_did : identifiant device Instagram.

Points à vérifier côté sécurité :

- Présence de l’attribut Secure
- Présence de l’attribut HttpOnly
- Présence de SameSite

Les cookies observés sont liés à la session ou au tracking, pas à un mot de passe.

---

## 5. Request Headers (extraits)

Analyse :

- User-Agent : identification de l’environnement (Android 10, Chrome 113).
- Origin et Referer : protections anti-CSRF.
- Content-Type : envoi de données en form-urlencoded.

Possibilité de fingerprinting via les headers.

---

## 6. Response Headers (extraits)

Analyse :

- 200 OK → requête acceptée.
- X-Frame-Options: DENY → protection clickjacking.
- Permissions-Policy → restriction API navigateur.
- CORP → protection cross-origin.

Présence de mécanismes de durcissement modernes.

---

## Conclusion

La requête analysée correspond à un mécanisme interne de reporting et non à une action critique.

Aucun élément sensible identifié dans :

- Query parameters
- Body parameters
- Cookies
- Headers

La cible applique plusieurs protections HTTP modernes.


