## 1️⃣ Choisir le scénario réel et fixer les objectifs

Pour un projet de fin de formation, il faut un cas d’usage qui :

* **Répond à un besoin concret** (ex. : assistance à la commande, réservation de voyage, information administrative).  
* **Permet de tester toutes les compétences acquises** : collecte de données, flux conversationnel, détection de langue, entraînement IA, déploiement multi‑canal et suivi analytics.  
* **Est réaliste pour une PME africaine** : budget limité, infrastructure web simple, forte utilisation mobile.

### Exemple de projet : « Boutique en ligne de mode « AfriStyle » »

* **Localisation** : Abidjan, Côte d’Ivoire.  
* **Produit** : vêtements et accessoires fabriqués localement.  
* **Canaux** : site WordPress, Facebook Messenger et WhatsApp Business.  
* **Langues cibles** : français, anglais, baoulé (langue locale) – le baoulé sera géré via traduction automatique.

**Objectifs mesurables**  

| KPI                     | Valeur cible (3 mois) |
|------------------------|-----------------------|
| Taux de résolution première réponse | ≥ 80 % |
| Temps moyen de réponse | ≤ 5 s |
| Satisfaction client (CSAT) | ≥ 4,5/5 |
| Nombre de conversations générées par le bot | 30 % du trafic total |

---

## 2️⃣ Collecte et structuration des questions fréquentes (FAQ)

### 2.1 Méthodologie de collecte

| Source | Action | Outil recommandé |
|--------|--------|------------------|
| Historique du service client (emails, tickets) | Export CSV | Google Sheets |
| Réseaux sociaux (messages privés) | Scraper via API Facebook | **voir chapitre 08** |
| Entretiens avec les agents | Questionnaire structuré | Google Forms |

### 2.2 Classification des intents

Utilisez la taxonomy suivante (exemple simplifié) :

| Intent | Exemple d’utterance | Réponse type |
|--------|----------------------|--------------|
| `order_status` | « Où est ma commande ? » | « Votre commande n°12345 est en cours de livraison et arrivera le 28/09. » |
| `size_guide` | « Quel taille dois‑je prendre ? » | Guide des tailles (PDF) |
| `return_policy` | « Comment retourner un article ? » | Procédure de retour en 3 étapes |
| `store_hours` | « Quand êtes‑vous ouverts ? » | Horaires d’ouverture |
| `language_switch` | « Parlez‑moi en anglais » | Changement de langue |

Sauvegardez le tableau sous forme de **JSON** compatible avec la plupart des plateformes no‑code :

```json
[
  {
    "intent": "order_status",
    "samples": [
      "Où est ma commande ?",
      "Quel est le statut de ma livraison ?",
      "Suivi de commande"
    ],
    "responses": {
      "fr": "Votre commande {order_id} est {status}.",
      "en": "Your order {order_id} is {status}.",
      "ba": "Mɛ́ {order_id} nɛ́ {status}."
    }
  }
]
```

---

## 3️⃣ Conception du flux conversationnel (voir chapitre 04)

### 3.1 Diagramme de haut niveau

```
[Start] → DetectLanguage → ChooseIntent → (Intent-specific block)
   ↓                     ↓
[Fallback] ← (No match) ← (Low confidence)
```

### 3.2 Gestion du multilinguisme

* **Détection** : appel à l’API Google Cloud Translation **detectLanguage** dès le premier message.  
* **Switch** : si l’utilisateur demande explicitement un changement de langue, on met à jour le **context** `user_language`.  
* **Traduction des réponses** : les réponses stockées en français sont traduites à la volée via **translateText** pour `en` et `ba`.  

### 3.3 Exemple de bloc « Suivi de commande »

1. **Prompt** : « Quel est votre numéro de commande ? »  
2. **Capture** : variable `order_id`.  
3. **Appel API interne** (voir § 5) pour récupérer le statut.  
4. **Réponse** : texte formaté avec la langue du contexte.

---

## 4️⃣ Implémentation de la logique serveur (Webhook)

Même sur une plateforme no‑code, le traitement de la logique métier (ex. : appel à l’ERP) se fait via un **webhook**. Voici un exemple minimal en **Node.js** (Express) hébergé sur **Render** (plan gratuit) :

```js
// server.js
const express = require('express');
const bodyParser = require('body-parser');
const fetch = require('node-fetch');
require('dotenv').config();

const app = express();
app.use(bodyParser.json());

// Endpoint appelé par le bot
app.post('/webhook', async (req, res) => {
  const { intent, entities, language } = req.body;
  let reply = {};

  if (intent === 'order_status') {
    const orderId = entities.order_id;
    // Simuler un appel à l’API interne
    const apiResp = await fetch(`${process.env.ERP_URL}/orders/${orderId}`);
    const data = await apiResp.json();

    const statusFr = `Votre commande ${orderId} est ${data.status_fr}.`;
    // Traduction dynamique si besoin
    if (language !== 'fr') {
      const tr = await translate(statusFr, language);
      reply.text = tr;
    } else {
      reply.text = statusFr;
    }
  } else {
    reply.text = 'Désolé, je n’ai pas compris votre demande.';
  }

  res.json(reply);
});

// Fonction de traduction (Google Cloud)
async function translate(text, targetLang) {
  const url = `https://translation.googleapis.com/language/translate/v2`;
  const params = new URLSearchParams({
    q: text,
    target: targetLang,
    key: process.env.GCP_KEY
  });
  const resp = await fetch(`${url}?${params}`);
  const json = await resp.json();
  return json.data.translations[0].translatedText;
}

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Webhook listening on ${PORT}`));
```

**Points à retenir**  

* Le webhook renvoie uniquement le texte ; la plateforme no‑code s’occupe du rendu UI.  
* Utilisez des variables d’environnement pour les clés API (sécurité).  
* Testez localement avec **ngrok** avant de déployer sur le cloud.

---

## 5️⃣ Entraînement du modèle IA (voir chapitre 06)

Pour les intents non couverts par les règles, on active le **LLM** (OpenAI gpt‑3.5‑turbo). Le prompt doit être **concis** et **inclure le contexte linguistique** :

```text
You are a multilingual customer support assistant for AfriStyle, an e‑commerce store in Côte d'Ivoire.
Respond in the language indicated by the variable {{lang}}.
Use only the information provided in the knowledge base below.

[Knowledge base]
- Shipping time: 2‑4 business days within Côte d'Ivoire.
- Return policy: 30 days with receipt.
- Size guide: link https://afristyle.ci/size-guide

User query: {{user_message}}
```

En pratique, on crée un **template** dans la plateforme no‑code qui injecte `{{lang}}` et `{{user_message}}` avant l’appel à l’API OpenAI. Le **temperature** est fixé à 0.2 pour garantir des réponses factuelles.

---

## 6️⃣ Phase de test et validation multilingue

### 6.1 Tests unitaires

| Test | Description | Résultat attendu |
|------|-------------|------------------|
| `detectLanguage` | Envoi de « How are you? » | `en` |
| `order_status` (FR) | Message « Quel est le statut de ma commande 12345 ? » | Réponse en français avec le bon statut |
| `order_status` (BA) | Message « Mɛ́ 12345 nɛ́ ? » | Réponse traduite en baoulé |

Utilisez **Postman** ou le **test console** de la plateforme pour automatiser ces scénarios.

### 6.2 Tests utilisateurs réels

1. **Recruter 5 personnes** (français, anglais, baoulé).  
2. **Scénario** : passer une commande fictive, demander le suivi, changer de langue.  
3. **Collecte de feedback** via un formulaire Google : note de satisfaction, compréhension, temps perçu.

Analysez les **KPIs** suivants :  

* **Precision de la langue** : % de réponses correctement traduites.  
* **Taux d’abandon** : % d’utilisateurs qui quittent avant le 2ᵉ message.  

---

## 7️⃣ Déploiement sur les canaux cibles

### 7.1 Intégration WordPress (voir chapitre 08)

1. **Installer le plugin “Insert Headers and Footers”**.  
2. **Copier le script du widget** fourni par la plateforme :

```html
<script src="https://cdn.chatbotplatform.com/widget.js" defer></script>
<script>
  window.ChatbotInit({
    botId: "afristyle-2024",
    containerId: "chatbot-widget",
    language: "auto"
  });
</script>
<div id="chatbot-widget"></div>
```

3. **Vérifier la compatibilité mobile** : le widget doit s’adapter aux écrans < 480 px.

### 7.2 Configuration WhatsApp Business API

* **Obtenir un numéro dédié** via un fournisseur local (ex. : Twilio, MessageBird).  
* **Créer le “WhatsApp Business Profile”** avec le logo d’AfriStyle et le lien vers le site.  
* **Connecter le webhook** (`/webhook`) dans le tableau de bord Twilio → “Sandbox → Messaging → Webhook URL”.  
* **Activer le “Message Template”** pour les notifications de suivi (ex. : `order_update`).

### 7.3 Publication sur Facebook Messenger

1. **Créer une page Facebook** pour AfriStyle.  
2. **Lier la page à la plateforme** via le “Facebook Messenger Channel”.  
3. **Autoriser le “Message Tag”** `CONFIRMED_EVENT_UPDATE` afin d’envoyer les confirmations de commande.  

---

## 8️⃣ Suivi analytics et itérations continues (voir chapitre 09)

### 8.1 Tableau de bord clé

| Métrique | Source | Fréquence |
|----------|--------|-----------|
| Sessions bot | Google Analytics (event `chatbot_open`) | Quotidienne |
| Intent le plus fréquent | Dashboard plateforme | Hebdomadaire |
| Taux de fallback | Logs webhook | Hebdomadaire |
| Satisfaction post‑chat | Survey link `https://forms.gle/…` | Après chaque session |

### 8.2 Boucle d’amélioration

1. **Identifier les intents à faible précision** (ex. : `size_guide` avec 60 % de fallback).  
2. **Enrichir le jeu de données** : ajouter 10 nouvelles utterances par langue.  
3. **Retrainer le modèle** (ou mettre à jour les règles) et redeployer.  
4. **Mesurer l’impact** : viser une amélioration de 15 % du taux de résolution.

---

## 9️⃣ Checklist de lancement

| ✅ | Action |
|----|--------|
| **1** | Finaliser la liste des intents et les réponses multilingues. |
| **2** | Déployer le webhook en production (HTTPS, certificat valide). |
| **3** | Configurer la détection de langue et les traductions dynamiques. |
| **4** | Tester chaque canal (site, WhatsApp, Messenger) en mode “Live”. |
| **5** | Activer le suivi analytics et le formulaire de satisfaction. |
| **6** | Former les agents internes à lire les rapports et à escalader les cas. |
| **7** | Mettre en place une alerte Slack/WhatsApp en cas d’erreur 5xx du webhook. |
| **8** | Publier un article de blog annonçant le nouveau service client IA. |
| **9** | Planifier la première revue de performance (30 jours après lancement). |

---

## 🔟 Monétisation et perspectives d’évolution

| Option | Description | Valeur ajoutée |
|--------|-------------|----------------|
| **Premium support** | Offrir un accès à un agent humain 24 h/24 via le même canal (tarif mensuel). | Augmente le revenu récurrent. |
| **Upsell automatisé** | Proposer des produits complémentaires après la confirmation de commande (ex. : “Complétez votre look avec…”). | Hausse du panier moyen. |
| **Data insights** | Vendre des analyses agrégées (trends de recherche, langues les plus utilisées) à des partenaires marketing. | Source de revenus B2B. |
| **Extension à d’autres langues** | Ajouter le swahili, le wolof ou le lingala selon la demande. | Accroît la portée géographique. |

---

## 📌 Points clés

* **Définir un objectif mesurable** dès le départ permet de quantifier le succès du bot.  
* **Structurer les FAQ** sous forme d’intents clairement nommés facilite la maintenance et l’entraînement du modèle.  
* **La détection de langue** doit être la première étape du flux ; la traduction dynamique évite de dupliquer les réponses.  
* **Le webhook** centralise la logique métier (consultation ERP, traduction) ; le sécuriser (HTTPS, clés) est indispensable.  
* **Les tests automatisés** (unitaires + utilisateurs) garantissent la robustesse avant le lancement public.  
* **Déployer sur plusieurs canaux** (site, WhatsApp, Messenger) maximise la visibilité du service client en Afrique, où le mobile est dominant.  
* **Un tableau de bord analytique** et une boucle d’amélioration continue transforment le bot d’un projet ponctuel en un actif évolutif.  
* **Monétiser** le chatbot (support premium, upsell, data) assure la viabilité économique pour la PME.  

En suivant ce fil conducteur, vous disposerez d’un chatbot IA multilingue pleinement fonctionnel, capable de répondre aux attentes des clients africains tout en générant de la valeur pour l’entreprise. Bon déploiement !