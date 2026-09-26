## Panorama des plateformes no‑code dédiées aux chatbots

Le marché des constructeurs de bots a explosé ces dernières années. Quatre acteurs se démarquent par leur maturité, leur communauté et leurs possibilités d’extension : **Chatfuel**, **ManyChat**, **Landbot** et **Voiceflow**. Chacun propose une approche « drag‑and‑drop », mais leurs spécificités diffèrent fortement lorsqu’il s’agit de gérer plusieurs langues et d’intégrer des modèles d’intelligence artificielle.

### Chatfuel  
* **Multilingue** : propose un bloc *Language Switcher* qui permet de rediriger l’utilisateur vers des flux distincts selon la langue détectée. La prise en charge native se limite à l’anglais, le français et l’espagnol ; pour d’autres langues (swahili, wolof…) il faut passer par des webhooks.  
* **IA** : intègre directement des blocs *AI* qui s’appuient sur le moteur de classification d’intentions de Chatfuel (basé sur Dialogflow). Pour du texte génératif (ex. réponses libres), il faut appeler une API externe (OpenAI, Cohere).  
* **Coût** : le plan gratuit autorise jusqu’à 50 000 messages/mois, mais les fonctions avancées (API, webhooks) sont réservées au plan *Pro* (à partir de 15 USD/mois).  
* **API / Webhooks** : accès complet aux webhooks HTTP, mais la documentation est en anglais.  
* **Communauté africaine** : présence d’un groupe francophone sur Facebook, mais peu d’événements locaux.

### ManyChat  
* **Multilingue** : propose un *Language Detection* intégré (basé sur le texte d’entrée) et la possibilité de créer des *Flows* distincts par langue. La prise en charge native inclut le français, l’anglais, l’arabe et le portugais.  
* **IA** : le *Smart Reply* s’appuie sur un modèle propriétaire. Pour des réponses plus poussées, ManyChat expose un *External Request* qui peut appeler n’importe quel endpoint REST.  
* **Coût** : le plan gratuit limite à 1 000 abonnés et 10 000 messages/mois. Le plan *Pro* débute à 10 USD/mois pour 5 000 abonnés, avec l’accès aux API.  
* **API / Webhooks** : très riche, avec des *JSON Templates* faciles à manipuler. La console propose un éditeur de code en ligne (Node.js).  
* **Communauté africaine** : plusieurs groupes francophones actifs, notamment au Kenya et au Sénégal, où les utilisateurs partagent des « templates » adaptés aux SMS et à WhatsApp.

### Landbot  
* **Multilingue** : la plateforme ne possède pas de détection automatique intégrée, mais elle offre un *Multilingual Block* qui permet de charger des traductions depuis un fichier JSON ou Google Sheet. Cette méthode est idéale pour des langues locales (haoussa, bambara).  
* **IA** : intègre nativement *OpenAI* et *Cohere* via des blocs *AI* qui envoient le texte de l’utilisateur à l’API et affichent la réponse. Aucun serveur intermédiaire n’est requis.  
* **Coût** : le plan gratuit permet 100 chatbots et 1 000 interactions/mois. Le plan *Starter* (15 USD/mois) débloque les intégrations API et la personnalisation du domaine.  
* **API / Webhooks** : très simple : chaque bloc possède un champ *Webhook URL* où l’on peut pousser ou récupérer des données. Les réponses sont renvoyées au format JSON.  
* **Communauté africaine** : forte présence en Côte d’Ivoire et au Maroc grâce à des ateliers organisés par *Empire du Web*.

### Voiceflow  
* **Multilingue** : conçu d’abord pour les assistants vocaux, il propose un *Language Switch* qui active des *Flows* distincts selon la langue détectée par le moteur de reconnaissance (Google Speech). La prise en charge du texte est secondaire mais fonctionnelle.  
* **IA** : dispose d’un *AI Block* compatible avec les modèles GPT‑3/4, ainsi que d’une intégration native à *Dialogflow* pour la classification d’intentions.  
* **Coût** : le plan gratuit offre 1 projet et 1 000 interactions/mois. Le plan *Pro* (25 USD/mois) donne accès aux API externes et aux exportations de code.  
* **API / Webhooks** : très complet, avec des *Variables* qui peuvent être envoyées à un endpoint et récupérées dans le flux. La documentation est disponible en français.  
* **Communauté africaine** : communauté grandissante au Nigeria et en Tunisie, avec des meet‑ups organisés par des incubateurs tech.

---

## Critères de sélection adaptés au contexte africain

Choisir la plateforme qui correspond à vos besoins ne se résume pas à comparer les prix. Voici les critères qui ont le plus d’impact pour les développeurs et les entrepreneurs francophones en Afrique.

### Coût et modèle d’abonnement  
Les projets de PME ou d’associations locales fonctionnent souvent avec des budgets limités. Il faut donc :

* **Évaluer le volume mensuel de messages** : les plans gratuits sont suffisants pour un pilote (ex. 5 000 messages/mois).  
* **Comparer le coût par abonné** : ManyChat facture par nombre d’abonnés, tandis que Landbot facture par interactions, ce qui peut être plus économique pour des campagnes saisonnières.  
* **Vérifier les frais de paiement** : certaines plateformes ne supportent que les cartes Visa/Mastercard, alors que les utilisateurs en Afrique utilisent largement le mobile money (M‑Pesa, Orange Money). Il faut donc s’assurer que la plateforme accepte les passerelles locales ou qu’elle permette d’intégrer un webhook de paiement.

### Support multilingue natif et extensibilité  
Le multilinguisme doit être fluide :

* **Détection automatique** : ManyChat et Chatfuel offrent une détection basique, mais pour les langues locales il faut passer par un service externe (ex. Google Cloud Translation).  
* **Gestion des traductions** : Landbot se distingue par son approche basée sur des fichiers JSON ou Google Sheets, ce qui simplifie la maintenance des textes traduits.  
* **Capacité à ajouter de nouvelles langues** : choisissez une plateforme où l’on peut facilement ajouter un nouveau bloc de traduction sans dupliquer tout le flux.

### Connectivité API et webhooks  
Un chatbot multilingue ne se limite pas à répondre ; il doit pouvoir :

* **Interroger un service de traduction** (Google, DeepL, ou un modèle open‑source hébergé sur un serveur local à Abidjan).  
* **Appeler un modèle IA** (OpenAI, Cohere, ou un modèle Hugging Face auto‑hébergé).  
* **Synchroniser les données client** avec votre CRM (Odoo, Zoho) via des requêtes REST.  

Landbot et Voiceflow offrent les webhooks les plus simples à configurer, tandis que Chatfuel nécessite parfois de passer par un *JSON API* plus verbeux.

### Accessibilité internet et performance  
Dans de nombreuses zones rurales, la bande passante est limitée :

* **Temps de réponse** : privilégiez une plateforme qui minimise les appels externes. Landbot, en intégrant directement OpenAI, réduit le nombre de sauts réseau.  
* **Mode hors‑ligne** : certains constructeurs permettent de pré‑charger les réponses fréquentes (FAQ) pour éviter les appels API à chaque interaction.

### Communauté locale et documentation en français  
Un support réactif est crucial :

* **Groupes d’entraide** : ManyChat et Landbot disposent de groupes Facebook actifs où les membres partagent des snippets de code adaptés aux opérateurs mobiles africains.  
* **Tutoriels vidéo en français** : Voiceflow propose une série de vidéos sous‑titrées, utiles pour les développeurs qui débutent.  
* **Événements locaux** : *Empire du Web* organise régulièrement des ateliers pratiques à Dakar, Nairobi et Kinshasa ; choisir une plateforme déjà présentée lors de ces sessions accélère la prise en main.

---

## Tableau comparatif synthétique

| Critère                | Chatfuel | ManyChat | Landbot | Voiceflow |
|------------------------|----------|----------|---------|-----------|
| Détection langue native| Oui (EN/FR/ES) | Oui (FR/EN/AR/PT) | Non (via JSON/Sheets) | Oui (via Speech) |
| IA générative intégrée | Via webhook | Via webhook | Direct OpenAI/Cohere | Direct OpenAI/Dialogflow |
| Webhooks / API         | ✅ (JSON) | ✅ (External Request) | ✅ (Simple URL) | ✅ (Variables) |
| Plan gratuit (messages) | 50 k | 10 k | 1 k | 1 k |
| Prix Pro (USD/mois)    | 15 | 10 | 15 | 25 |
| Paiement mobile‑money  | ❌ (via intégration) | ✅ (via webhook) | ✅ (via webhook) | ✅ (via webhook) |
| Communauté francophone | Modérée | Active | Très active | Active |
| Idéal pour langues locales | ⚠️ | ⚠️ | ✅ | ⚠️ |

---

## Créer un compte test sur Landbot (exemple pratique)

Landbot se révèle souvent le meilleur compromis entre **facilité d’intégration**, **coût** et **support des langues locales**. Suivez ces étapes pour disposer rapidement d’un environnement de test.

### Inscription étape par étape  

1. **Accéder au site** : rendez‑vous sur https://landbot.io et cliquez sur **“Start for free”**.  
2. **Choisir le mode “No‑code”** : Landbot propose deux produits (No‑code & Bot‑builder). Sélectionnez **No‑code**.  
3. **S’inscrire avec une adresse e‑mail** : utilisez une adresse professionnelle (ex. `contact@myshop.sn`). Un e‑mail de confirmation vous sera envoyé.  
4. **Valider le compte** : cliquez sur le lien de vérification. Vous êtes alors redirigé vers le tableau de bord.  
5. **Configurer le profil** : indiquez le nom de votre entreprise (ex. *Boutique Dakar*), le secteur (e‑commerce) et la langue principale (français).  

### Vérification du compte  

Landbot demande parfois une **vérification d’identité** pour débloquer les intégrations tierces :  

* **Téléverser une pièce d’identité** (CNI ou passeport).  
* **Attendre 24 h** pour l’approbation.  

Une fois le compte validé, vous avez accès à la **bibliothèque de templates** et à la **console de test**.

---

## Configurer un bot de base multilingue sur Landbot

Le but de cet exercice est de créer un petit assistant qui :

1. Accueille l’utilisateur en français ou en anglais.  
2. Détecte la langue via un webhook qui interroge l’API **Google Cloud Translation Detect**.  
3. Envoie la requête à **OpenAI** pour générer une réponse adaptée.  
4. Retourne la réponse traduite dans la langue d’origine.

### Définir le flux d’accueil  

1. **Bloc “Welcome”** : texte « Bonjour ! / Hello! » avec deux boutons : **Français** et **English**.  
2. **Bloc “Set Language”** : créez une variable `{{lang}}` qui prend la valeur `fr` ou `en` selon le bouton cliqué.  
3. **Bloc “User Input”** : champ texte libre où l’utilisateur pose sa question.  

### Ajouter la détection de langue via webhook  

Même si l’utilisateur a choisi une langue, il est prudent de vérifier la langue du texte saisi (ex. un client francophone peut taper en anglais). Créez un bloc **“Detect Language”** avec le webhook suivant :

```json
POST https://translation.googleapis.com/language/translate/v2/detect
Content-Type: application/json
Authorization: Bearer YOUR_GOOGLE_API_KEY

{
  "q": "{{user_input}}"
}
```

Le webhook renvoie :

```json
{
  "data": {
    "detections": [
      [
        {
          "language": "en",
          "confidence": 0.99,
          "isReliable": true
        }
      ]
    ]
  }
}
```

Dans Landbot, mappez la réponse à la variable `{{detected_lang}}` :

* `{{detected_lang}} = response.data.detections[0][0].language`

Ensuite, ajoutez une **condition** : si `{{detected_lang}} != {{lang}}`, alors mettez à jour `{{lang}}` avec la langue détectée. Ainsi, le bot s