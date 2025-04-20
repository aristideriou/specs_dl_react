# specs_dl_wordpress

Exemple de specs pour un site Wordpress. Si jamais vous tombez un peu par hasard sur ce repo, n'hésitez pas à lire [cet article](https://www.mightandmetrics.io/blog/ecrire_specs) qui explique le pourquoi du comment de la démarche.

## 1 - Snippet GTM et règles CSP - Web

### CSP

Les directives [Content Security Policy suivantes](https://developers.google.com/tag-manager/web/csp) doivent être mises en place afin de prermettre la bonne exécution des tags depuis GTM : 

### Snippet GTM

Insérer ce code aussi haut que possible dans le ```<head>``` de la page : 

```javascript
<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-PF7MN30');</script>
```

Insérer ce code aussi haut que possible dans le ```<body>``` :

```javascript
<noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-PF7MN30"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
```

Pour valider la bonne installation de GTM, suivre les étapes suivantes : 

- S'assurer que l'on a bien supprimé / désactivé tout plugin qui pourrait bloquer le déclenchement de Google Tag Manager (ex : uBlock) 
- Vérifier l'appel au fichier gtm.js dans le navigateur, répond en 200 : 

## Calcul du data layer server side / Locomotive

### Alimentation du data layer au chargement d'une page en mode loggué

Lorsqu'une page est chargée depuis Locomotive, et que l'app React est rechargée, *uniquement lorsque l'utilisateur est connecté* : 

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'userLogin',
  'userLogin':{
    'id':'123456789', //String : ID utilisateur. 
    'status':'Individuel' //String : "Individuel" ou "Entreprise", si le compte dispose d'un parent entreprise
  }
});
```

Idéalement, ce bloc de code doit être poussé **avant** l'appel à Google Tag Manager (cf. partie suivante).

## Appel des snippets GTM et Axeptio

Au chargement d'une page généré depuis Locomotive, insérer ce code aussi haut que possible dans la balise ```<head>``` de la page : 

```javascript
<script>
window.axeptioSettings = {
 clientId: '6075aa35afa7d303f949b7f9',
};
 
(function(d,s) {
 var t = d.getElementsByTagName(s)[0], e = d.createElement(s);
 e.async = true; e.src = '//static.axept.io/sdk.js';
 t.parentNode.insertBefore(e, t);
})(document, 'script');
</script>
```

Insérer ce code aussi haut que possible dans le ```<head>``` de la page : 

```javascript
<!-- Google Tag Manager -->
<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-MWBFZ22');</script>
<!-- End Google Tag Manager -->
```

Insérer ce code aussi haut que possible dans le ```<body>``` :

```javascript
<!-- Google Tag Manager (noscript) -->
<noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-MWBFZ22"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
<!-- End Google Tag Manager (noscript) -->
```

Si les librairies JS sont appelées via un package manager comme Yarn ou NPM [(exemple ici)](https://www.npmjs.com/package/@analytics/google-tag-manager), le résultat doit être le même en front : il suffit en général d'appeler GTM avec l'ID 'GTM-MWBFZH8' : 

```bash
npm install analytics
npm install @analytics/google-tag-manager
```

```javascript
import Analytics from 'analytics'
import googleTagManager from '@analytics/google-tag-manager'

const analytics = Analytics({
  app: 'my-app-name',
  plugins: [
    googleTagManager({
      containerId: 'GTM-MWBFZH8'
    })
  ]
})
```

## Alimentations de GTM client side

Les interactions suivantes avec le data layer se font au sein d'une page / route, sans rechargement, et sont donc en principe purement gérées côté JS / React.

### Chargement d'une route produit (affichage de l'encart produit sur une page de marque)

L'ensemble des informations poussées dans le data layer provient du JSON "produit" retourné par l'ERP, :

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'productRouteLoad',
  'product':{
    'name':'HA0 - Salade de crozets, panais et carottes au comté A.O.P. (Avec pain)',//String : "_source.Name" du JSON
    'variant':'Avec pain',//String : "variant_attributes.gestion_des_inclus" du JSON
    'price' :50,//Integer : "price.default.original_value_taxed" du JSON
    'sku':'HA0A',//String : "_source.sku" du JSON.
    'brand':'LECLERC',//String : "Brand.name" du JSON
    'priceCategory':'€€€',//String : "attributes.tarif_" du JSON
    'diet':['Végétarien','Sans porc']//Array de strings : "attributes.régimes_alimentaires" du JSON
  }
});
```

### Ajout au panier ou retrait d'un produit 

Lors de l'ajout / retrait d'un produit au panier ou la modification d'une quantité ("Encart fiche produit" / "Fiche produit" / "Checkout") :

Il s'agit uniquement de renseigner la quantité **ajoutée** : si un utilisateur a déjà 2 produits dans son panier, et en ajoute 4, ne renseigner que les informations sur les produits ajoutés, pas sur l'ensemble du panier.

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'cartChange',
  'cartChange':{
    'type':'Ajout',//String : "Ajout" ou "Retrait"
    'position':'Encart fiche produit',//String : "Encart fiche produit" / "Page marque" / "Checkout",
    'quantity':3//Integer : nombre de produits ajoutés
  },
  'productsAddedToCart' : [{//Même logique que lors du chargement d'une vue produit (cf. ci-dessus)
    'name':'HA0 - Salade de crozets, panais et carottes au comté A.O.P. (Avec pain)',//String - "_source.Name" du JSON
    'variant':'Avec pain',//String : "variant_attributes.gestion_des_inclus" du JSON
    'price' :50,//Integer : "price.default.original_value_taxed" du JSON
    'sku':'HA0A',//String : "_source.sku" du JSON.
    'brand':'LECLERC',//String : "Brand.name" du JSON
    'priceCategory':'€€€',//String : "attributes.tarif_" du JSON
    'diet':['Végétarien','Sans porc']//Array : "attributes.régimes_alimentaires" du JSON
  },
  {
    'name':'HA0 - Salade de crozets, panais et carottes au comté A.O.P. (Avec pain)',
    'variant':'Avec pain',/
    //...
    //etc, 1 objet par produit ajouté / retiré 
  }]
});
```

### Chargement des routes du checkout (hors validation commande)

Au chargement de chaque étape du checkout et de l'affichage de la vue dédiée : Panier, Livraison et facturation, Paiement :

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'checkoutStepLoad',
  'checkoutStepLoad':{
    'name':'Panier',//String : nom du step : "Panier", "Livraison et facturation", "Paiement"
    'id':3//Integer : Numéro du step
  }
});
```

### Validation de commande

Au moment où la transaction est validée, le callback de la plateforme de paiement reçu, et le message de confirmation affiché :

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'orderConfirmation',
  'orderConfirmation':{
    'id':'123456789',//String : ID de la commande
    'priceHT':200,//Integer : Prix de la commande 
    'taxes':10,//Integer : TVA (si pertinent)
    'deliveryDate':20220221,//Integer : date à laquelle la commande sera livrée au format AAAAMMJJ
    'postCode':'75001',//String : Code postal de la ville
    'products':[{//Array : un item par produit acheté
      'name':'HA0 - Salade de crozets, panais et carottes au comté A.O.P. (Avec pain)',//String - "_source.Name" depuis le JSON
      'variant':'Avec pain',//String : "variant_attributes.gestion_des_inclus" du JSON
      'sku':'HA0A',//String : "_source.sku" du JSON.
      'brand':'DUPONT',//String : "Brand.name" depuis le JSON
    },
    {
      'name':'HA1, XXXXX',//String - "_source.Name" depuis le JSON
      'variant':'XXX',//String : "variant_attributes.gestion_des_inclus" du JSON
      'sku':'HA99',//String : "_source.sku" du JSON.
      'brand':'XXXX',//String : "Brand.name" depuis le JSON
    },
    {
      //etc, un item par produit commandé
    }
  ]} 
});
```

### Interactions avec la popin de saisie de l'adresse / date créneau

Lors de l'ouverture de la pop in ou du clic sur le bouton de recherche : 

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'productAddPopin',
  'productAddPopin':{
    'interaction':'Ouverture',//String : Type d'interaction : "Ouverture" ou "Recherche"
    'context':'Homepage'//String : vaut "Homepage", "Formulaire header", "Popin" 
});
```

### Validation demande de devis en ligne

Lorsque la demande est validée (affichage d'une popin, route dédiée...)

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'quoteValidation',
  'quoteValidation':{
    'postCode':'75001',//String : Code postal de la ville
    'guests':4,//Integer : Nombre d'invités
    'eventType':'Petit-déjeuner',//String : Type d'événement sélectionné dans le formulaire ("Petit déjeuner", "Cocktail apéritif"...)
    'date':'2022-04-02'//String : date souhaitée au format AAAA-MM-JJ
  }
});
```

### Clic sur le numéro de téléphone

Lorsque l'utilisateur clique sur un des 3 emplacements indiquant le numéro de téléphone (ouvrant l'application téléphone ou équivalente de l'utilisateur) : 

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'phoneNumberClick',
  'phoneNumberClick':{
    'position':'Header',//String : Emplacement du bouton : "Header", "Réassurance", "Pop In"
  }
});
```

### Validation de l'abonnement newsletter

Lorsque la demande est validée (affichage d'une popin, route dédiée...)

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'newsletterSubscription'
});
```

### Validation d'un des formulaires de contact

Validation de l'un des 3 formulaires présents sur le site : Presse, Contact, Devenir Partenaire

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'formContact',
  'formContact':{
    'type':'Contact',//String : "Contact", "Presse", "Devenir Partenaire"
    'topic':'Etre recontacté',//String : Motif du formulaire renseigné par l'utilisateur : "Etre recontacté", "Faire une réclamation"...
  }
});
```

### Utilisations de la barre de recherche instantannée

3 types d'interaction doivent être mesurées : Le clic sur la barre de recherche, l'affichage d'un résultat suite à la réponse de l'API, et le clic d'un utilisateur sur un résultat de recherche

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'searchBarInteraction',
  'searchBarInteraction':{
    'userAction':'Clic barre de recherche',//String : "Clic barre de recherche", "Affichage résultat", "Clic résultat"
    'nbResults':3,//Integer : nombre de résultats affichés, uniquement dans le cas de "Affichage résultat"
  }
});
```

### Affichage d'un résultat de recherche

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'searchPageLoad',
  'searchPageLoad':{
    'nbResults':3,//Integer : nombre de résultats affichés, uniquement dans le cas de "Affichage résultat"
    'city':'Paris',//String : nom de la ville recherchée, sans code postal
    'date':'2022-04-02',//String : date recherchée au format AAAA-MM-JJ
    'timeslot':'10h30 - 11h30'//String : créneau recherché au format "heure début - heure fin" 
  }
});
```

### Chargement d'une page de marque

L'ensemble des informations poussées dans le data layer provient du JSON "marque" retourné par l'ERP :

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'brandPageLoad',
  'brandPageLoad':{
    'brandName':'Fauchon',//String : "_source.name" du JSON
    'priceCategory':'€€€',//String : "_source.price_range" du JSON
    'productTags':['Boissons','Plateaux repas']//Array de strings : "_source.product_tags.name" du JSON
  }
});
```

### Clic sur un filtre de la page de recherche

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'searchFilterClick',
  'searchFilterClick':{
    'name':'Sans Porc',//String : nom du filtre tel qu'il est affiché sur la page : "Sans porc", "Végétarien"... 
  }
});
```

### Chargement d'un contenu du blog

```javascript
window.dataLayer = window.dataLayer || [];
dataLayer.push({
  'event':'contentLoad',
  'contentLoad':{
    'publicationDate':1642941251,//Integer : Timestamp UNIX en secondes de la date de publication du contenu
    'tags':['Cuisine','Recette'],//Array de strings : Liste des tags associés au contenu
    'contentLength':693//Integer : nombre de signes de l'article
  }
});
```

## 6. Décoration d'éléments HTML

Certains éléments HTML de différents composants devront comporter des paramètres "data-tms-XXXX" afin que Google Tag Manager puisse capter certaines actions utilisateur liées (entrée dans le viewport, clic, focus...)

### Homepage

Les section suivantes de la homepage doivent comporter une balise ```<data-tms-homepage>``` :

Partenaires : ```<data-tms-homepage="Partenaires">```

Actus : ```<data-tms-homepage="Actus">```

Footer : ```<data-tms-homepage="Footer>```

### Pages produits

La section suivante des pages produit doit comporter une balise ```<data-tms-product>``` :

Sélection produit : ```<data-tms-product="Sélection produit">```

### Contenus

La section suivants des pages de contenus (blog) doit comporter une balise ```<data-tms-content>``` :

Pied d'article : ```<data-tms-content="Pied d'article">```

## 6. Dépendances techniques

Certains comportements de l'app sont utilisés pour pousser des éléments d'analytics, et doivent rester consistants dans le temps. En cas de changement, informer les contacts indiqués en début de document :

### Routes JS

Google Tag Manager capture les changements d'URL "virtuelle" (faits via un "pushState" en JS) afin d'envoyer un tag de page vue à Google Analytics. Toute route appelée de cette façon et n'étant pas pertinente (ex : route technique) doit être communiquée et discutée.

## Pages 404

Afin de mesurer le nombre de chargement de pages 404, et sachant qu'il n'est pas possible de récupérer le code de réponse d'une page, le pattern de la balise ```<title>``` doit être consistant, et valoir exactement "Page non trouvée". Tout changement doit être communiqué.
