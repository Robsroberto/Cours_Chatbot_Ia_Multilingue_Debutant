## Insertion du widget sur un site WordPress  

### Choisir le mode d’intégration  

| Méthode | Avantages | Inconvénients |
|--------|-----------|---------------|
| **Plugin dédié** (ex. *Chatbot for WP*) | Installation en un clic, mise à jour automatique, options de style dans le tableau de bord. | Dépendance à un tiers, parfois limité aux plateformes de chatbot supportées. |
| **Code HTML/JavaScript** | Liberté totale sur le design, fonctionne avec n’importe quel service no‑code. | Nécessite un accès au thème ou à l’éditeur de blocs. |
| **Bloc Gutenberg personnalisé** | Intégration native dans l’éditeur, possibilité de réutiliser le bloc sur plusieurs pages. | Un peu plus technique, demande la création d’un petit plugin ou d’un snippet. |

Pour les développeurs débutants en Afrique francophone, le **code HTML/JavaScript** est souvent le plus simple car il ne dépend pas d’un plugin qui pourrait ne plus être maintenu. La plupart des plateformes no‑code (Landbot, ManyChat, Chatfuel) génèrent un script d’insertion que l’on copie‑colle dans le site.

### Étapes d’insertion  

1. **Récupérer le script** depuis la console de votre plateforme chatbot.  
   Exemple (Landbot) :  

   ```html
   <script src="https://static.landbot.io/landbot-3/landbot-3.0.0.js"></script>
   <script>
     var myLandbot = new Landbot.Livechat({
       configUrl: 'https://chats.landbot.io/v3/H-1234567/index.json',
     });
   </script>
   ```

2. **Ouvrir le tableau de bord WordPress** → *Apparence* → *Éditeur de thème* (ou *Personnaliser* → *CSS/JS additionnels*).  

3. **Coller le script** dans le fichier `footer.php` juste avant la balise `</body>` ou, si vous utilisez un thème enfant, créer un fichier `footer-custom.php` et l’inclure.  

   ```php
   <?php
   // footer-custom.php
   ?>
   <!-- Début du widget chatbot -->
   <script src="https://static.landbot.io/landbot-3/landbot-3.0.0.js"></script>
   <script>
     var myLandbot = new Landbot.Livechat({
       configUrl: 'https://chats.landbot.io/v3/H-1234567/index.json',
     });
   </script>
   <!-- Fin du widget chatbot -->
   <?php wp_footer(); ?>
   ```

4. **Sauvegarder** et vérifier sur le front‑end que le bouton apparaît en bas à droite.  

### Personnalisation du style  

- **Couleurs** : la plupart des plateformes offrent un paramètre `theme` dans le script.  
- **Position** : ajouter `position: 'left'` ou `position: 'bottom'` dans les options.  
- **Déclencheur** : pour les connexions mobiles lentes, désactiver l’ouverture automatique et laisser l’utilisateur cliquer.  

```js
var myLandbot = new Landbot.Livechat({
  configUrl: 'https://chats.landbot.io/v3/H-1234567/index.json',
  theme: { primaryColor: '#0066cc' },
  position: 'right',
  openOnLoad: false
});
```

---

## Déploiement sur Wix  

### Utiliser le **HTML iFrame**  

1. Dans l’éditeur Wix, cliquer sur **+** → **Intégrer** → **HTML iframe**.  
2. Coller le même script que pour WordPress, mais encapsulé dans une balise `<div>` avec l’attribut `id`.  

   ```html
   <div id="my-chatbot"></div>
   <script src="https://static.landbot.io/landbot-3/landbot-3.0.0.js"></script>
   <script>
     new Landbot.Livechat({
       configUrl: 'https://chats.landbot.io/v3/H-1234567/index.json',
       container: document.getElementById('my-chatbot')
     });
   </script>
   ```

3. Ajuster la taille du conteneur (ex. `width="350"` `height="500"`).  

### Astuce pour les visiteurs à bande passante limitée  

Wix autorise le chargement différé (`lazy load`). Activez‑le dans les paramètres du bloc HTML afin que le script ne soit chargé que lorsqu’il entre dans le viewport.

---

## Configuration sur WhatsApp Business  

### Créer un compte Business API  

1. **S’inscrire** sur le portail Meta for Developers → *WhatsApp Business API*.  
2. Obtenir un **numéro de téléphone** dédié (préférablement un numéro local).  
3. Générer un **token d’accès** (Bearer token) qui sera utilisé par le middleware du chatbot.  

### Relier le numéro à la plateforme no‑code  

- **ManyChat** : dans *Settings* → *Channels* → *WhatsApp*, coller le token et choisir le numéro.  
- **Chatfuel** : ajouter un *WhatsApp Plugin* et fournir le `Phone ID` et le `Access Token`.  

### Gestion du flux conversationnel  

Le flux doit commencer par un **message de bienvenue** qui indique la langue disponible. Exemple de script ManyChat :

```json
{
  "messages": [
    {
      "text": "👋 Bienvenue ! Vous pouvez parler en français, anglais ou swahili. Quelle langue préférez‑vous ?"
    },
    {
      "quick_replies": [
        { "title": "Français", "payload": "LANG_FR" },
        { "title": "English", "payload": "LANG_EN" },
        { "title": "Kiswahili", "payload": "LANG_SW" }
      ]
    }
  ]
}
```

Le choix du payload déclenche le **router multilingue** (voir chapitre 5).  

### Optimisation pour les réseaux mobiles africains  

- **Message de pré‑validation** : avant d’envoyer le premier message, demander l’accord de l’utilisateur pour recevoir des notifications (conformité RGPD).  
- **Compression** : activer la compression des médias (images, vidéos) via l’option `media_url` de l’API afin de réduire la consommation de données.  
- **Tarification locale** : privilégier les messages texte courts (< 160 caractères) pour éviter les frais supplémentaires sur les opérateurs.  

---

## Publication sur Facebook Messenger  

### Étapes de mise en place  

1. **Créer une page Facebook** dédiée à votre service client (ou utiliser une page existante).  
2. **Activer Messenger** dans les *Paramètres* → *Plateforme Messenger*.  
3. **Obtenir le Page Access Token** via le *Graph API Explorer*.  
4. **Configurer le webhook** (URL qui recevra les événements) dans le tableau de bord développeur.  

   Exemple de webhook Node.js (Express) :

   ```js
   const express = require('express');
   const bodyParser = require('body-parser');
   const app = express();
   app.use(bodyParser.json());

   app.post('/webhook', (req, res) => {
     const entry = req.body.entry[0];
     const messaging = entry.messaging[0];
     const senderId = messaging.sender.id;
     const message = messaging.message?.text || '';

     // Appeler le moteur IA (voir chapitre 6) et renvoyer la réponse
     handleMessage(senderId, message);
     res.sendStatus(200);
   });

   app.listen(process.env.PORT || 3000);
   ```

5. **Vérifier** le webhook avec le token de vérification fourni par Facebook.  

### Adaptation aux contraintes africaines  

- **Limite de 20 000 messages/mois** pour les comptes gratuits : prévoir un plan de montée en charge dès que le volume dépasse ce seuil.  
- **Langues** : Facebook Messenger supporte le *language code* dans le payload (`"locale":"fr_FR"`). Assurez‑vous d’envoyer le bon code en fonction du choix de l’utilisateur.  
- **Boutons rapides** : privilégier les *quick replies* plutôt que les *carousel cards* qui consomment plus de bande passante.  

---

## Spécificités des connexions mobiles en Afrique  

### Gestion des numéros courts  

Dans plusieurs pays (ex. Côte d’Ivoire, Sénégal), les services client utilisent des **numéros courts** (ex. 1234). Pour intégrer ces canaux :  

- **USSD Gateway** : sous‑traiter à un opérateur qui expose une API USSD.  
- **Exemple d’appel API** (via `axios`) :

  ```js
  const axios = require('axios');
  const ussdEndpoint = 'https://api.operator.com/ussd/send';

  async function sendUssd(sessionId, message) {
    await axios.post(ussdEndpoint, {
      session_id: sessionId,
      text: message,
      short_code: '1234'
    }, {
      headers: { Authorization: `Bearer ${process.env.OPERATOR_TOKEN}` }
    });
  }
  ```

- **Flux conversationnel** : chaque réponse du chatbot doit être courte (≤ 160 caractères) pour éviter le découpage en plusieurs SMS.  

### Optimisation du temps de réponse  

- **Cache côté serveur** : stocker les réponses aux questions fréquentes (FAQ) dans Redis pour éviter un appel IA à chaque fois.  
- **Détection de la bande passante** : le script du widget peut mesurer la vitesse de connexion (`navigator.connection.downlink`). Si la vitesse < 0.5 Mbps, désactiver les animations et proposer un mode texte simplifié.  

  ```js
  if (navigator.connection && navigator.connection.downlink < 0.5) {
    myLandbot.update({ openOnLoad: false });
  }
  ```

### Compatibilité avec les téléphones bas de gamme  

- **Design responsive** : éviter les éléments CSS lourds (shadows, gradients) qui ralentissent les rendus sur les appareils Android 5.x.  
- **Polices système** : privilégier `sans-serif` natif plutôt que des polices Google Fonts.  

---

## Gestion des droits d’accès et conformité RGPD  

### Rôles et permissions  

| Rôle | Accès autorisé |
|------|----------------|
| **Administrateur** | Gestion complète du bot, modification du flux, accès aux analytics. |
| **Opérateur support** | Consultation des conversations en temps réel, possibilité d’intervenir via le *live chat*. |
| **Analyste** | Lecture seule des tableaux de bord, export de données anonymisées. |

Sur les plateformes no‑code, créez ces rôles dans la section *Team* ou *Members* et attribuez‑leur les scopes correspondants (`read:analytics`, `write:flows`).  

### Collecte et stockage des données personnelles  

- **Consentement explicite** : dès le premier message, demander l’accord (`"Acceptez‑vous que nous stockions vos données pour améliorer le service ?"`).  
- **Délai de rétention** : configurer une purge automatique après 30 jours (ou selon la législation locale). La plupart des plateformes offrent un paramètre *Data Retention* dans les réglages de la base de données.  

### Exportation et droit à l’oubli  

Fournir un **endpoint** simple que l’utilisateur peut appeler pour demander la suppression de ses données :

```js
app.delete('/api/user/:userId', async (req, res) => {
  const { userId } = req.params;
  await deleteUserData(userId); // supprime de la DB et du cache
  res.json({ status: 'deleted' });
});
```

Inclure le lien vers cet endpoint dans le message d’accueil (`"Envoyez /forget pour supprimer vos données"`).  

---

## Tests de mise en production  

| Test | Objectif | Méthode |
|------|----------|---------|
| **Test de charge** | Vérifier que le serveur supporte 500 conversations simultanées. | Utiliser `k6` ou `Locust` avec des scénarios multilingues. |
| **Test de compatibilité mobile** | S’assurer que le widget s’affiche correctement sur Android 6 et iOS 11. | Emuler avec Chrome DevTools → *Device Mode*. |
| **Test de conformité** | Valider que le consentement RGPD est enregistré. | Simuler un utilisateur qui refuse et vérifier l’absence de stockage. |

---

## Points clés  

- **Intégration directe** : le script fourni par la plateforme no‑code suffit pour WordPress et Wix ; adaptez la taille et le déclencheur selon la connexion de l’utilisateur.  
- **Canaux locaux** : WhatsApp Business et Facebook Messenger restent les plus répandus en Afrique, mais les numéros courts et USSD sont indispensables pour les zones à faible connectivité.  
- **Optimisation mobile** : détecter la bande passante, désactiver les animations lourdes, limiter la longueur des messages, et compresser les médias.  
- **Sécurité & RGPD** : mettre en place un consentement explicite, des rôles d’accès granulaire, une politique de rétention et un mécanisme de droit à l’oubli facilement accessible.  
- **Tests avant lancement** : charge, compatibilité mobile et conformité légale doivent être validés pour éviter les interruptions de service et les sanctions.  

En suivant ces bonnes pratiques, votre chatbot IA multilingue sera prêt à servir vos clients sur tous les points de contact numériques, même dans les environnements mobiles les plus contraints d’Afrique francophone.