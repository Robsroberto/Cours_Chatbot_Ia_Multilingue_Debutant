## Stratégie de test globale

Un chatbot qui fonctionne bien en laboratoire peut rapidement se heurter à des imprévus lorsqu’il est mis à l’épreuve par de vrais clients. La clé d’un déploiement fiable réside dans une **approche de test en plusieurs couches** :

| Niveau | Objectif | Exemple d’activité |
|--------|----------|--------------------|
| **Unitaires** | Vérifier que chaque bloc (détection d’intent, appel API, génération de texte) renvoie le résultat attendu. | Tests automatisés sur les fonctions de traduction (voir chapitre 5). |
| **Intégration** | S’assurer que les différents blocs s’enchaînent correctement dans le flux conversationnel. | Simuler un scénario de réclamation qui passe par la détection de langue → appel du modèle IA → enregistrement du ticket. |
| **Scénarios réels** | Valider le comportement du bot face à des conversations naturelles, incluant des fautes d’orthographe, du code‑switching ou des expressions locales. | Sessions de test avec des agents du service client ou des bêta‑testeurs. |
| **Performance** | Mesurer le temps de réponse, le taux de résolution et la satisfaction utilisateur. | Utiliser les dashboards d’analytics intégrés (voir chapitre 9). |
| **A/B / Canary** | Comparer deux versions de réponse (ex. : formulation courte vs. détaillée) sur un sous‑ensemble de trafic. | Déployer la version « A » à 10 % des visiteurs et la version « B » aux 90 % restants. |

Cette matrice permet de **couvrir le spectre complet** : du code isolé aux interactions en production, tout en gardant un œil sur les indicateurs business.

---

## Tests unitaires : automatiser le cœur du bot

### Pourquoi les tests unitaires sont indispensables

- **Détection précoce des régressions** : lorsqu’une mise à jour du modèle IA modifie la forme des réponses, les tests signalent immédiatement les écarts.
- **Documentation vivante** : chaque fonction testée devient une forme de spécification exécutable.
- **Facilite le refactoring** : vous pouvez améliorer le code sans crainte de briser une fonctionnalité déjà validée.

### Mise en place avec une plateforme no‑code

Même si la plateforme est no‑code, la plupart offrent une **API REST** ou un **SDK** permettant d’appeler les blocs de logique. Voici un exemple avec **Landbot** (JavaScript) :

```javascript
// test/detectLanguage.test.js
const { detectLanguage } = require('../src/detectLanguage');
const assert = require('assert');

describe('Détection de langue', () => {
  it('devrait identifier le français même avec fautes d’orthographe', async () => {
    const input = "Bonjou, je veux savoir mon solde";
    const result = await detectLanguage(input);
    assert.strictEqual(result, 'fr');
  });

  it('devrait retourner "en" pour un texte anglais', async () => {
    const input = "Hello, I need help with my order";
    const result = await detectLanguage(input);
    assert.strictEqual(result, 'en');
  });
});
```

Ce test s’appuie sur la fonction `detectLanguage` qui, dans le projet, encapsule l’appel à l’API Google Cloud Translation (voir chapitre 5). En exécutant `npm test`, vous obtenez un feedback immédiat.

### Bonnes pratiques

| Pratique | Raison |
|----------|--------|
| **Nommer les tests de façon explicite** | Facilite le diagnostic lorsqu’un test échoue. |
| **Mocker les appels externes** | Évite les dépendances réseau et rend les tests rapides et déterministes. |
| **Couvrir les cas limites** | Ex. : texte vide, caractères spéciaux, langues rares (swahili, haoussa). |
| **Intégrer les tests dans le pipeline CI/CD** | Chaque push déclenche le suite de tests (GitHub Actions, GitLab CI). |

---

## Tests d’intégration : le flux complet en action

### Simuler un parcours client

Un test d’intégration doit reproduire **l’enchaînement** des blocs du bot : réception du message, détection de langue, appel du modèle IA, génération de la réponse, mise à jour du CRM. Voici un scénario typique de réclamation de livraison :

1. **Message client** : « Mon colis n’est pas arrivé ».
2. **Détection de langue** → `fr`.
3. **Appel du modèle IA** avec le prompt : « Réponds en français, empathiquement, en demandant le numéro de suivi. ».
4. **Enregistrement du ticket** dans le CRM via webhook.
5. **Réponse au client** : « Bonjour, désolé pour ce désagrément. Pouvez‑vous me communiquer votre numéro de suivi ? »

### Exemple de script d’intégration (cURL + jq)

```bash
#!/bin/bash
# test/integration/complaint_flow.sh

# 1. Envoi du message au webhook du bot
response=$(curl -s -X POST https://api.landbot.io/v1/webhooks/CHATBOT_ID \
  -H "Content-Type: application/json" \
  -d '{"message":"Mon colis n’est pas arrivé"}')

# 2. Extraction de la réponse du bot
bot_reply=$(echo "$response" | jq -r '.reply')
echo "Bot répond : $bot_reply"

# 3. Vérification du contenu de la réponse
if [[ "$bot_reply" == *"numéro de suivi"* ]]; then
  echo "✅ Scénario OK"
  exit 0
else
  echo "❌ Scénario KO – réponse inattendue"
  exit 1
fi
```

Ce script peut être intégré à un job GitHub Actions :

```yaml
name: Integration tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run integration test
        run: bash test/integration/complaint_flow.sh
```

### Gestion des environnements

- **Sandbox** : utilisez un environnement de test isolé (clé API dédiée) pour éviter d’impacter les vraies conversations.
- **Données factices** : créez des tickets fictifs dans le CRM afin que les webhooks ne polluent pas la base de production.

---

## Tests en conditions réelles : bêta‑testeurs et crowdsourcing

### Recruter des testeurs locaux

Le contexte africain introduit des **variantes linguistiques** (français de Côte d’Ivoire, français du Sénégal, code‑switching avec le wolof ou le lingala). Organisez une session de bêta‑test avec :

- **Agents du service client** : ils connaissent les questions fréquentes et les réponses attendues.
- **Clients volontaires** : via un formulaire Google ou un groupe WhatsApp dédié.

### Collecte de feedback qualitatif

1. **Enregistrement des conversations** (respect de la RGPD et des lois locales sur la protection des données).  
2. **Questionnaire post‑interaction** : « La réponse était‑elle claire ? », « Le temps d’attente était‑il raisonnable ? ».  
3. **Analyse des écarts** : comparez les réponses attendues (script de FAQ) avec les réponses générées.

### Outils de test en ligne

- **Postman** : créez des collections d’appels API pour chaque scénario et partagez‑les avec l’équipe.
- **Botium** : framework dédié aux tests de chatbot, compatible avec la plupart des plateformes no‑code. Il permet de déclarer des **conversations de test** sous forme de fichiers **.txt** :

```
# Test de prise de rendez‑vous
User: Bonjour, je veux prendre un rendez‑vous pour un visa.
Bot: Bien sûr, dans quel pays souhaitez‑vous faire votre demande ?
User: Côte d’Ivoire
Bot: Parfait, quelle date vous convient le mieux ?
```

Botium exécutera ces dialogues et signalera les écarts.

---

## Indicateurs de performance (KPIs) à surveiller

### 1. Taux de résolution (Resolution Rate)

> **Formule** : (Nombre d’interactions où le problème est résolu) ÷ (Nombre total d’interactions) × 100 %

- **Résolution première réponse** : mesure la capacité du bot à répondre correctement dès le premier échange.
- **Escalade** : pourcentage d’interactions transférées à un agent humain. Un taux d’escalade trop élevé indique que le bot ne comprend pas assez de cas.

### 2. Temps moyen de réponse (Average Response Time)

- **Latency technique** : temps entre la réception du message et la réponse du serveur (ms).  
- **Latency perçue** : temps que l’utilisateur estime attendre (s).  
- **Objectif** : < 2 s de latence technique, < 5 s de latence perçue pour un service client fluide.

### 3. Satisfaction utilisateur (CSAT)

- **Score à 5 étoiles** ou **NPS** (Net Promoter Score) intégré à la conversation : après chaque interaction, le bot propose : « Comment avez‑vous trouvé cette réponse ? ».  
- **Analyse sémantique** : utilisez le NLP (voir chapitre 2) pour détecter les mots « merci », « pas utile », etc., dans les messages libres.

### 4. Taux d’abandon (Drop‑off Rate)

- **Définition** : proportion d’utilisateurs qui quittent la conversation avant d’obtenir une réponse.  
- **Signal d’alerte** : pics d’abandon pendant les étapes de validation (ex. : demande de numéro de suivi).

### 5. Couverture multilingue

- **Pourcentage de conversations traitées dans chaque langue**.  
- **Taux d’erreur de détection** : proportion de messages où la langue a été mal identifiée (voir chapitre 5).

---

## Exploiter les dashboards d’analytics intégrés

La plupart des plateformes no‑code offrent un tableau de bord **en temps réel**. Voici comment le configurer pour le suivi quotidien :

| Widget | Métrique | Paramètres recommandés |
|--------|----------|------------------------|
| **Graphique ligne** | Temps moyen de réponse | Filtrer par jour, par langue |
| **Camembert** | Taux d’escalade | Séparer par type de demande (FAQ, réclamation, prise de rendez‑vous) |
| **Heatmap** | Volume d’interactions par heure | Identifier les pics d’activité (ex. : après les heures de bureau) |
| **Liste** | Top 10 des intentions non reconnues | Exporter pour enrichir le jeu de données (voir chapitre 6) |

### Alertes automatisées

- **Seuil de latence** : déclencher un webhook Slack si le temps moyen dépasse 5 s.  
- **Taux d’escalade > 20 %** : ouvrir un ticket JIRA automatiquement pour enquêter.

Exemple de configuration d’alerte sur **ManyChat** :

```json
{
  "trigger": "average_response_time",
  "condition": "greater_than",
  "value": 5000,
  "action": {
    "type": "webhook",
    "url": "https://hooks.slack.com/services/T000/B000/XXXX"
  }
}
```

---

## Optimisation continue : du feedback aux itérations

### 1. Analyse des “intentions non reconnues”

- **Export CSV** des logs d’intentions non mappées.  
- **Clusterisation** : utilisez un petit script Python (sklearn) pour regrouper les phrases similaires et identifier des nouvelles intentions récurrentes.

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.cluster import KMeans
import pandas as pd

df = pd.read_csv('unmatched_intents.csv')
vectorizer = TfidfVectorizer(stop_words='french')
X = vectorizer.fit_transform(df['message'])

kmeans = KMeans(n_clusters=5, random_state=42).fit(X)
df['cluster'] = kmeans.labels_
print(df.groupby('cluster')['message'].apply(list).head())
```

- **Mise à jour du modèle** : ajoutez les nouvelles phrases à votre jeu de données d’entraînement (voir chapitre 6) puis ré‑entraînez le modèle.

### 2. Raffinement des réponses

- **A/B testing de formulations** : créez deux variantes de réponse (ex. : courte vs. détaillée) et mesurez le CSAT.  
- **Personnalisation locale** : intégrez des expressions idiomatiques propres à chaque pays (ex. : « c’est bon ? » au Sénégal).  
- **Gestion des fautes d’orthographe** : enrichissez le dictionnaire de correction avec des termes courants (ex. : « téléphone » → « telfono » en lingala).  

### 3. Boucle de ré‑entraînement automatisée

1. **Collecte** : chaque jour, récupérer les conversations où le CSAT ≤ 3.  
2. **Annotation** : via une tâche interne ou une plateforme de crowdsourcing (ex. : Appen).  
3. **Enrichissement** : ajouter les nouvelles paires *question‑réponse* au dataset.  
4. **Déploiement** : déclencher un pipeline CI qui entraîne le modèle (OpenAI fine‑tuning ou Hugging Face) et le pousse automatiquement sur la plateforme.

```yaml
# .github/workflows/retrain.yml
name: Retrain Model
on:
  schedule:
    - cron: '0 2 * * *'   # chaque nuit à 02h00 UTC
jobs:
  retrain:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Prepare data
        run: python scripts/prepare_training_data.py
      - name: Fine‑tune model
        run: python scripts/fine_tune.py --api-key ${{ secrets.OPENAI_KEY }}
      - name: Deploy to platform
        run: curl -X POST https://api.landbot.io/v1/models/deploy -H "Authorization: Bearer ${{ secrets.LANDBOT_TOKEN }}" -d '{"model_id":"my_finetuned"}'
```

### 4. Gestion du multilinguisme en production

- **Détection de langue** : surveillez le taux d’erreur par langue. Un pic d’erreur en **haoussa** peut indiquer un