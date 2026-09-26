## Préparer vos jeux de données : FAQ, scripts de support et formats compatibles  

Un modèle de génération de texte n’apprend que ce que vous lui fournissez.  
Pour un chatbot service client, les sources les plus courantes sont :  

| Source | Exemple concret | Format recommandé |
|--------|----------------|-------------------|
| FAQ du site web | « Quel est le délai de livraison ? » | CSV ou JSON, une ligne par question‑réponse |
| Scripts d’appels téléphoniques | « Bonjour, je souhaite modifier mon abonnement. » | TSV (tab‑separated) avec **intent** et **utterance** |
| Tickets de support (Zendesk, Freshdesk) | « Mon paiement a été refusé. » | JSONL (une ligne JSON) contenant **question**, **answer**, **langue** |

### Nettoyage et structuration  

1. **Uniformiser les langues** – ajoutez un champ `langue` (`fr`, `en`, `sw`, `yo`…) afin que le modèle puisse être interrogé dans la bonne langue.  
2. **Supprimer les doublons** – un même libellé répété plusieurs fois biaise le modèle.  
3. **Normaliser la ponctuation** – évitez les espaces avant les points, uniformisez les guillemets.  

```python
import pandas as pd
import unicodedata

df = pd.read_csv('faq_raw.csv')
# Normalisation Unicode
df['question'] = df['question'].apply(lambda x: unicodedata.normalize('NFKC', x))
df['answer']   = df['answer'].apply(lambda x: unicodedata.normalize('NFKC', x))

# Suppression des doublons
df = df.drop_duplicates(subset=['question', 'langue'])

df.to_json('faq_clean.jsonl', orient='records', lines=True)
```

Le fichier `faq_clean.jsonl` pourra être importé directement dans les plateformes no‑code (ex. : **Bubble** via le plugin “CSV/JSON uploader”) ou dans les APIs d’OpenAI / Cohere.

---

## Choisir le bon fournisseur IA et créer un “fine‑tuning” sans code  

### OpenAI : Fine‑tuning de `gpt‑3.5‑turbo`  

OpenAI propose une interface web où l’on téléverse un fichier JSONL contenant les paires **prompt / completion**.  
Exemple de ligne :

```json
{
  "messages": [
    {"role": "system", "content": "Tu es un assistant service client multilingue."},
    {"role": "user",   "content": "Quel est le coût de la livraison à Bamako ?"}
  ],
  "completion": "La livraison à Bamako coûte 2 000 FCFA et prend 3 à 5 jours ouvrés."
}
```

#### Étapes no‑code  

| Étape | Action | Outil |
|------|--------|-------|
| 1 | Créez un compte OpenAI, accédez à **Fine‑tuning** | Tableau de bord OpenAI |
| 2 | Importez le fichier `faq_clean.jsonl` | Bouton “Upload file” |
| 3 | Sélectionnez le modèle de base (`gpt‑3.5‑turbo`) | Menu déroulant |
| 4 | Lancez le job de fine‑tuning | “Create fine‑tune” |
| 5 | Récupérez l’ID du modèle (`ft-xxxx`) pour l’utiliser dans votre chatbot no‑code | API key + modèle ID |

### Cohere : Fine‑tuning de `command‑r`  

Cohere fonctionne de façon similaire, mais accepte les fichiers CSV.  
Exemple de ligne CSV :

```csv
prompt,completion,language
"Comment puis‑je récupérer mon mot de passe ?", "Cliquez sur « Mot de passe oublié » puis suivez les instructions.", "fr"
```

Sur la console Cohere, choisissez **Create Custom Model**, téléversez le CSV, puis définissez le **temperature** et le **max tokens**.

### Hugging Face : Entraînement d’un modèle open‑source (ex. : `mistralai/Mistral-7B-Instruct`)  

Pour les organisations qui souhaitent garder le contrôle total, Hugging Face propose des espaces de **AutoTrain**. Aucun script Python n’est requis ; il suffit de :

1. Créer un **Dataset** à partir du même JSONL.  
2. Sélectionner le modèle de base.  
3. Configurer les hyper‑paramètres (learning‑rate, epochs).  
4. Lancer l’entraînement.  

Le modèle entraîné est alors disponible via une **Inference API** que vous pouvez appeler depuis votre plateforme no‑code (ex. : **Zapier** → “Webhooks – POST”).

---

## Concevoir des prompts efficaces : le secret d’une génération fiable  

Un bon prompt agit comme un cadre qui oriente le modèle. Voici trois patterns éprouvés pour le service client.

### 1. Prompt « system + user » (OpenAI)  

```json
{
  "messages": [
    {"role": "system", "content": "Tu es un assistant service client pour une boutique en ligne de mode africaine. Réponds en français, en restant poli et concis."},
    {"role": "user",   "content": "Je n’ai pas reçu ma commande, que faire ?"}
  ]
}
```

*Pourquoi ?* Le rôle **system** fixe le ton et le domaine, réduisant les dérives.

### 2. Prompt « instruction + input » (Cohere)  

```
Instruction: Réponds à la question suivante en français, en moins de 30 mots.
Input: Quel est le statut de ma facture du 12/08/2024 ?
```

Le séparateur `Instruction / Input` clarifie la tâche et le modèle se concentre sur la génération de la réponse.

### 3. Prompt « few‑shot examples » (Hugging Face)  

```
Q: Quels sont les modes de paiement acceptés ?
A: Nous acceptons les cartes Visa, Mastercard, ainsi que Mobile Money (MTN, Orange).

Q: Comment suivre ma livraison ?
A: Connectez‑vous à votre compte, cliquez sur « Suivi de commande » et entrez le numéro fourni.

Q: {question_utilisateur}
A:
```

En fournissant deux exemples, le modèle infère le format attendu.

---

## Ajuster la température et le nombre de tokens : trouver le juste équilibre  

| Paramètre | Effet | Valeur typique (service client) |
|-----------|-------|---------------------------------|
| **temperature** | Variabilité de la réponse ; 0 = déterministe, 1 = créatif | 0.2 – 0.4 |
| **max_tokens** | Longueur maximale de la réponse | 150 – 250 (suffisant pour des réponses complètes) |
| **top_p** | Contrôle de la probabilité cumulative (alternative à temperature) | 0.9 (par défaut) |

**Exemple d’appel API (OpenAI) :**

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "model": "ft-xxxxxxxxxxxx",
        "messages": [
          {"role":"system","content":"Tu es un assistant service client multilingue."},
          {"role":"user","content":"Comment annuler ma réservation ?"}
        ],
        "temperature": 0.3,
        "max_tokens": 180,
        "frequency_penalty": 0.0,
        "presence_penalty": 0.0
      }'
```

Dans les plateformes no‑code, ces paramètres sont généralement exposés sous forme de **sliders** (ex. : le composant “OpenAI Action” de **Make.com**).

---

## Tester les réponses en plusieurs langues : protocole de validation  

### 1. Créer un jeu de tests multilingue  

| ID | Langue | Question | Réponse attendue (exemple) |
|----|--------|----------|----------------------------|
| T01 | fr | “Quel est le tarif d’un retour produit ?” | “Le retour est gratuit sous 30 jours, sinon 5 % du prix.” |
| T02 | en | “How long does shipping to Lagos take?” | “Shipping to Lagos takes 2‑4 business days.” |
| T03 | sw | “Ninaweza kupokea pesa yangu?” | “Pesa yako itarejeshwa ndani ya siku 7.” |

Sauvegardez ce tableau au format **CSV** pour le charger dans votre outil de test automatisé.

### 2. Automatiser le test avec **Zapier**  

1. **Trigger** : “Schedule” (exécution quotidienne).  
2. **Action** : “Code by Zapier” – boucle sur chaque ligne du CSV, envoie la requête à l’API du modèle.  
3. **Action** : “Filter” – compare la réponse avec le champ *Réponse attendue* (utilisation de `levenshtein` ou `contains`).  
4. **Action** : “Slack” – notifie l’équipe si le taux de réussite chute sous 90 %.

```javascript
// Exemple de code Zapier (JavaScript)
const fetch = require('node-fetch');

const testCases = inputData.test_cases; // tableau d'objets
let failures = [];

for (const tc of testCases) {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      model: process.env.FINE_TUNED_MODEL,
      messages: [
        {role: 'system', content: 'Tu es un assistant service client multilingue.'},
        {role: 'user',   content: tc.question}
      ],
      temperature: 0.3,
      max_tokens: 180
    })
  });
  const data = await response.json();
  const answer = data.choices[0].message.content.trim();

  if (!answer.toLowerCase().includes(tc.expected.toLowerCase())) {
    failures.push({id: tc.id, question: tc.question, answer, expected: tc.expected});
  }
}

return {failures};
```

### 3. Interpréter les résultats  

- **Taux de réussite ≥ 95 %** : le modèle est prêt pour la mise en production.  
- **80 % ≤ taux < 95 %** : identifier les intents problématiques, enrichir le dataset avec des variantes.  
- **taux < 80 %** : envisager un deuxième cycle de fine‑tuning ou ajuster la température (réduire la créativité).

---

## Adapter le modèle à des langues locales : stratégies pratiques  

1. **Enrichir le dataset avec des traductions** – utilisez des traducteurs automatiques (Google Cloud Translation) puis faites valider les traductions par un locuteur natif.  
2. **Utiliser des modèles multilingues** – `gpt‑3.5‑turbo` comprend déjà plus de 100 langues, mais pour des langues peu représentées (ex. : **wolof**, **bambara**) il peut être utile de **pré‑entraîner** un petit modèle sur des corpus locaux (ex. : articles de presse en wolof).  
3. **Détecter la langue avant l’appel IA** – la plupart des plateformes no‑code offrent un bloc “Detect Language” (ex. : **Landbot**). Le résultat (`lang`) détermine quel **endpoint** appeler :  
   - `lang = fr` → modèle fine‑tuned français  
   - `lang = en` → modèle fine‑tuned anglais  
   - `lang = other` → modèle général multilingue  

```json
{
  "detect_language": "fr",
  "model_id": "ft-fr-12345"
}
```

4. **Post‑traitement** – pour les langues à orthographe complexe (ex. : **haoussa** avec caractères diacritiques), appliquez un correcteur orthographique via l’API **LanguageTool** avant d’envoyer la réponse à l’utilisateur.

---

## Intégrer le modèle fine‑tuned dans votre chatbot no‑code  

### Exemple avec **Bubble**  

| Élément | Configuration |
|--------|----------------|
| API Connector | Créez une API “OpenAI‑FT” : <br>**URL** : `https://api.openai.com/v1/chat/completions` <br>**Headers** : `Authorization: Bearer <your_key>` |
| Paramètres | `model = ft-xxxx` <br> `temperature = 0.3` <br> `max_tokens = 200` |
| Workflow | 1️⃣ L’utilisateur saisit sa question → 2️⃣ Action “Detect Language” → 3️⃣ Appel API “OpenAI‑FT” avec le prompt structuré → 4️⃣ Affichage de la réponse dans le groupe de chat. |

### Exemple avec **ManyChat** (via **Make.com**)  

1. **Trigger** : “New Message” dans ManyChat.  
2. **Module HTTP** : POST vers l’endpoint OpenAI, corps JSON incluant le champ `messages` construit à partir du texte reçu.  
3. **Router** : selon la valeur `language` du message, choisir le modèle (`ft-fr-…` ou `ft-en-…`).  
4. **Return** : envoyer la réponse à ManyChat avec l’action “Send Text”.

---

## Mettre à jour le modèle sans repartir de zéro  

Les besoins évoluent : nouveaux produits, nouvelles politiques, ou changement de ton.  

1. **Collecte continue** – exportez les conversations réelles (exclure les PII) toutes les semaines.  
2. **Étiquetage semi‑automatique** – utilisez le modèle actuel pour proposer une réponse, puis laissez l’agent humain corriger. Le couple *question / correction* constitue un nouveau **example**.  
3. **Fine‑tuning incrémental** – OpenAI accepte des jeux de données additionnels via l’option **“Add data to existing fine‑tune”**. Cela évite de repartir de zéro et réduit le coût.  
4. **Versionnage** – nommez chaque itération (`ft-fr-v2`, `ft-fr-v3`). Conservez les anciens modèles comme fallback en cas de régression.

---

## Sécurité et conformité des données  

- **Masquage des données sensibles** : avant d’envoyer des tickets à l’API, supprimez les numéros de carte, les identifiants clients, etc.  
- **Chiffrement** : stockez les jeux de données sur **AWS S3** ou **Azure Blob** avec chiffrement côté serveur.  
- **RGPD & LGPD locales** : conservez les logs de traitement pendant la durée légale (ex. : 12 mois) et assurez