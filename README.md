# Lab3 >> BurpSuite

---

## 1. Preparation de BurpSuite

### HTTP history est present 
<p align="center"> <img src="images/z1.png" width="500"> </p>

--- 
## Intercept is off

<p align="center"> <img src="images/z2.png" width="600"> </p>

---
## 2. Checking Proxy Listener:

### Port du proxy : 8080
### Adresse d’écoute : 127.0.0.1 (Loopback only)
<p align="center"> <img src="images/z3.png" width="600"> </p>

## Interprétation
### 127.0.0.1 (Loopback only)

- Le proxy écoute uniquement sur la machine locale.
- Utilisé lorsque le navigateur est sur la même machine que Burp.
- Configuration adaptée pour tests locaux.

### All interfaces (0.0.0.0)

- Le proxy écoute sur toutes les interfaces réseau.
- Utilisé lorsque :
  - Un émulateur Android externe est utilisé
  - Un appareil mobile réel est connecté
- Nécessite configuration réseau correcte (IP locale + port).
---
## 3. Adresse de la machine hote :
<IP_HOTE> : 10.0.2.2
<p align="center"> <img src="images/z4.png" width="300"> </p>
<p align="center"> <img src="images/z6.png" width="400"> </p>

---

## 4. Proxy configuration :
<p align="center"> <img src="images/z5.png" width="300"> </p>

---




## 5. Analyse du trafic HTTP dans Burp Suite 
<p align="center"> <img src="images/z7.png" width="800"> </p>
<p align="center"> <img src="images/z8.png" width="400"> </p>
<p align="center"> <img src="images/z9.png" width="800"> </p>
<p align="center"> <img src="images/z12.png" width="700"> </p>

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


# 6. Test 1: Analyse d’une requête HTTP interceptée (Instagram AJAX)

<p align="center"> <img src="images/z10.png" width="700"> </p>
<p align="center"> <img src="images/z11.png" width="300"> </p>

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

## 5. Request Headers 

Analyse :

- User-Agent : identification de l’environnement (Android 10, Chrome 113).
- Origin et Referer : protections anti-CSRF.
- Content-Type : envoi de données en form-urlencoded.

Possibilité de fingerprinting via les headers.

---

## 6. Response Headers 

Analyse :

- 200 OK → requête acceptée.
- X-Frame-Options: DENY → protection clickjacking.
- Permissions-Policy → restriction API navigateur.
- CORP → protection cross-origin.

Présence de mécanismes de durcissement modernes.

---


# Test 2 : OWASP Juice Shop — Analyse d’une requête d’authentification (Burp Suite)

<p align="center"> <img src="images/z13.png" width="700"> </p>
<p align="center"> <img src="images/zz7.png" width="700"> </p>

## Contexte

- Cible : https://demo.owasp-juice.shop
- Outil : Burp Suite (Proxy → HTTP history)
- Objectif : Lecture et analyse d’une requête HTTP en mode observation (sans modification)
- Requête analysée : `POST /rest/user/login` (tentative de connexion)

---

## 1) Lecture dans l’onglet Raw (Request)

### Méthode

- `POST`  

### Chemin et paramètres

- Chemin : `/rest/user/login`  
Interprétation : endpoint REST d’authentification (login).

- Paramètres d’URL : aucun paramètre significatif observé sur cette requête  
Interprétation : les données sont envoyées dans le corps (body), pas dans l’URL.

### En-têtes (headers) observés et interprétation

Les headers exacts peuvent varier selon le navigateur/émulateur. Les plus importants à analyser sont :

- `Host: demo.owasp-juice.shop`  
  Confirme la cible.

- `Content-Type: application/json`  
  Le body est un JSON.

- `Accept: application/json, text/plain, */*`  
  Le client accepte principalement du JSON.

- `User-Agent: Mozilla/5.0 (Linux; Android ...) Chrome/...`  
  Identifie l’environnement (Android + Chrome). Utile pour comprendre le contexte et le fingerprinting.

- `Origin: https://demo.owasp-juice.shop`  
  Indique l’origine de la requête. Souvent utilisé côté serveur pour contrôler CSRF / CORS.

- `Referer: https://demo.owasp-juice.shop/#/login`   
  Page depuis laquelle la requête a été déclenchée. Utile pour la traçabilité du flux.



### Corps de la requête (body)

Exemple observé (format typique Juice Shop) :

`json
{
  "email": "user@example.com",
  "password": "********"
}`  






# 7. Burp Suite — Interception temporaire d’une requête HTTP


<p align="center"> <img src="images/zz1.png" width="400"> </p>
<p align="center"> <img src="images/zz2.png" width="600"> </p>
<p align="center"> <img src="images/zz3.png" width="600"> </p>
<p align="center"> <img src="images/zz4.png" width="600"> </p>
<p align="center"> <img src="images/zz5.png" width="600"> </p>

## Objectif

Observer une requête HTTP en temps réel sans la modifier, puis la laisser continuer vers le serveur.

---

## Étapes réalisées

### 1. Activation de l’interception

Dans :

```
Proxy → Intercept
```

Cliquer sur :

```
Intercept is on
```

Interprétation :

- Burp intercepte désormais les requêtes avant qu’elles n’atteignent le serveur.
- Toute nouvelle requête sera mise en attente.

---

### 2. Déclenchement d’une requête

Dans le navigateur :

- Rafraîchir une page autorisée (ex: http://httpbin.org/get)

Effet observé :

- La requête apparaît dans l’onglet Intercept.
- Elle est affichée comme "en attente".
- Le navigateur semble bloqué (chargement en cours).

Exemple observé :

```
GET http://httpbin.org/get
```

---

### 3. Observation de la requête interceptée (Raw)

Éléments analysés :

- Méthode : GET
- URL : /get
- Host : httpbin.org
- Headers : User-Agent, Accept, Accept-Encoding, etc.

À ce stade :

- La requête n’a pas encore été envoyée au serveur.
- Aucune réponse n’a été reçue.

---

### 4. Relâcher la requête

Deux options :

- Cliquer sur `Forward` → la requête est envoyée au serveur.
- Cliquer sur `Drop` → la requête est supprimée.

Après avoir cliqué sur `Forward` :

- La page se charge normalement.
- La requête apparaît dans `HTTP history`.

---

### 5. Désactiver l’interception

Cliquer sur :

```
Intercept is off
```

Pourquoi ?

- Éviter de bloquer toutes les requêtes suivantes.
- Revenir en mode observation passive.

---

# 8.  Installation du certificat dans l’émulateur Android
- Permet d’ajouter une autorité de certification personnalisée.
- Nécessaire pour que le navigateur fasse confiance aux certificats générés dynamiquement par Burp.


## Compréhension du mécanisme

Sans certificat CA installé :

- L’émulateur ne fait pas confiance à Burp.
- Les connexions HTTPS échouent.
- Aucun trafic HTTPS exploitable.

Avec certificat CA installé :

- Le navigateur accepte les certificats générés par Burp.
- Le trafic HTTPS devient visible dans Burp.
- Analyse complète possible (headers, body, tokens, etc.).
  
<p align="center"> <img src="images/h1.png" width="400"> </p>
<p align="center"> <img src="images/h2.png" width="400"> </p>
<p align="center"> <img src="images/h3.png" width="400"> </p>
<p align="center"> <img src="images/h4.png" width="400"> </p>
<p align="center"> <img src="images/h5.png" width="400"> </p>
<p align="center"> <img src="images/h6.png" width="400"> </p>
<p align="center"> <img src="images/h7.png" width="400"> </p>
<p align="center"> <img src="images/h8.png" width="400"> </p>



# Mini-rapport d’analyse — Interception HTTP/HTTPS en environnement de labo

---

## 1. Périmètre

- Environnement : Émulateur Android (laboratoire isolé)
- Cible : Application de test autorisée (ex. OWASP Juice Shop)
- Objectif : Observation du trafic HTTP/HTTPS sans modification
- Nature du test : Analyse passive

---

## 2. Configuration du test

- Outil : Burp Suite Community Edition
- Version : <VERSION_BURP>
- Machine hôte (IP locale) : <IP_HOTE>
- Port proxy : <PORT_PROXY>
- Type de listener : Loopback only / All interfaces
- Date : <DATE>
- Heure : <HEURE>
- Certificat CA installé : Oui (certificat Burp dans l’émulateur)

---

## 3. Preuves (Evidence)

### 3.1 Capture de l’historique HTTP

Exemples de requêtes observées :

```
GET /get HTTP/1.1
Host: httpbin.org
Status: 200 OK
```

```
POST /rest/user/login HTTP/2
Host: demo.owasp-juice.shop
Status: 401 Unauthorized
```

---

### 3.2 Détail d’une requête analysée

#### Requête :

```
POST /rest/user/login HTTP/2
Host: demo.owasp-juice.shop
Content-Type: application/json
Origin: https://demo.owasp-juice.shop
User-Agent: Mozilla/5.0 (Linux; Android ...)
```

Body :

```json
{
  "email": "user@example.com",
  "password": "********"
}
```

#### Réponse :

```
HTTP/2 401 Unauthorized
Content-Type: application/json
```

---

## 4. Analyse technique

### 4.1 Données observées en transit

- Paramètres JSON : email, password
- Headers : User-Agent, Origin, Referer
- Cookies éventuels : langue, bannière, session (selon état)
- Transport : HTTPS (TLS actif)

### 4.2 Risques potentiels identifiés

Observé :

- Données d’authentification envoyées en JSON (normal en HTTPS).
- Pas de données sensibles dans l’URL.
- Statut 401 en cas d’échec → comportement cohérent.

À surveiller dans un contexte réel :

- Tokens présents dans l’URL (risque de fuite via logs)
- Cookies sans attribut Secure / HttpOnly
- Absence d’en-têtes de sécurité HTTP
- Stockage local non sécurisé côté Android

---

## 5. Recommandations défensives

### 5.1 Côté serveur

- Utiliser HTTPS obligatoire
- Définir cookies avec :
  - Secure
  - HttpOnly
  - SameSite=Strict
- Éviter les tokens dans les URLs
- Implémenter rate limiting sur login
- Masquer les messages d’erreur trop explicites

### 5.2 Côté client Android

- Ne pas stocker tokens en clair
- Utiliser Android Keystore si nécessaire
- Activer Network Security Config
- Minimiser les données envoyées

### 5.3 Minimisation des données

- Ne transmettre que les champs strictement nécessaires
- Supprimer les paramètres inutiles
- Limiter l’exposition des identifiants techniques

---

## 6. Conclusion

Le test démontre la capacité à :

- Intercepter et analyser le trafic HTTP/HTTPS
- Identifier les données sensibles en transit
- Comprendre le contexte technique
- Documenter les observations de manière reproductible

Aucune vulnérabilité critique observée dans ce laboratoire.

---

## 7. Distinction importante

Ce rapport distingue clairement :

### Observé
- Requête POST vers endpoint d’authentification
- Données JSON contenant email/password
- Réponse 401 cohérente

### Supposé
- Mécanisme de validation serveur standard
- Gestion session via token après succès (non observé ici)

### Recommandé
- Renforcement des cookies
- Minimisation des données
- Bonnes pratiques Android

---

## Reproductibilité

Toute personne disposant :

- Du même lab
- De la même configuration proxy
- Du certificat installé

peut reproduire le test en suivant les étapes décrites.
