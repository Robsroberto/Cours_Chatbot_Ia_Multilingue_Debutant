## Détection automatique de la langue du client  

Les utilisateurs d’un service client africain peuvent écrire en français, en anglais, mais aussi dans des langues locales comme le swahili, le wolof ou le haoussa. La première étape d’un chatbot multilingue consiste donc à identifier **quelle** langue a été utilisée avant de choisir le modèle IA ou la réponse à générer.  

### 1. Services de détection de langue prêts à l’emploi  

| Service | Langues supportées | Mode d’appel | Tarif (exemple) |
|---------|-------------------|--------------|-----------------|
| **Google Cloud Translation – DetectLanguage** | +120 langues, incluant swahili (sw), wolof (wo), haoussa (ha) | REST / gRPC | 20 $ / M caractères détectés |
| **Azure Cognitive Services – Translator Text** | +70 langues, même couverture pour les langues africaines | REST | 10 $ / M caractères |
| **Amazon Translate – DetectDominantLanguage** | +55 langues, support limité pour le wolof | REST | 15 $ / M caractères |

Ces API renvoient généralement un **code ISO‑639‑1** (ex. `fr`, `en`, `sw`) accompagné d’un score de confiance. Le code pourra être directement utilisé pour :

* sélectionner le modèle de génération (ex. un modèle français vs un modèle anglais) ;
* choisir le dictionnaire de traduction locale si la langue n’est pas prise en charge par le service cloud ;
* enregistrer la langue dans les logs d’analyse (voir chapitre 9).  

### 2. Implémentation côté serveur (Node.js)  

Voici un exemple minimal qui utilise **Google Cloud Translation** pour détecter la langue, puis déclenche le bon flux :

```js
// detection.js
const {TranslationServiceClient} = require('@google-cloud/translate').v3;
const client = new TranslationServiceClient();

async function detectLanguage(text, projectId, location = 'global') {
  const request = {
    parent: `projects/${projectId}/locations/${location}`,
    content: text,
    mimeType: 'text/plain',
  };
  const [response] = await client.detectLanguage(request);
  const {languageCode, confidence} = response.languages[0];
  return {lang: languageCode, confidence};
}

// Exemple d’utilisation dans le routeur du chatbot
app.post('/webhook', async (req, res) => {
  const userMessage = req.body.message;
  const {lang, confidence} = await detectLanguage(userMessage, 'my-gcp-project');

  // seuil de confiance : 0,8 → on accepte la détection, sinon on bascule sur le français par défaut
  const selectedLang = confidence > 0.8 ? lang : 'fr';

  // stockage de la langue pour la session
  req.session.lang = selectedLang;

  // appel du moteur IA approprié (voir chapitre 6)
  const reply = await generateReply(userMessage, selectedLang);
  res.json({reply});
});
```

*Le code ci‑dessus montre :*  

* comment appeler l’API de détection ;  
* comment appliquer un **seuil de confiance** ;  
* comment persister la langue dans la session pour les prochains messages.  

### 3. Gestion des langues peu couvertes : dictionnaires personnalisés  

Certaines langues locales (ex. wolof) ne bénéficient pas d’une traduction de qualité via les services cloud. La solution consiste à créer un **dictionnaire de correspondance** : phrase en langue locale → phrase en français (ou anglais).  

#### 3.1. Structure du dictionnaire  

```json
// dictionaries/wolof.json
{
  "Nanga def?": "Comment ça va ?",
  "Jamm rekk": "Tout va bien",
  "Maa ngi ci jamm": "Je suis en paix",
  "Naka la xarit bi?": "Comment est votre ami ?"
}
```

#### 3.2. Algorithme de recherche floue  

Pour éviter que l’utilisateur doive écrire exactement la même phrase, on utilise une recherche de similarité (ex. Levenshtein). Exemple en Python :

```python
# wolof_fallback.py
import json, difflib

with open('dictionaries/wolof.json') as f:
    DICO = json.load(f)

def translate_wolof(sentence):
    # recherche de la phrase la plus proche
    best_match = difflib.get_close_matches(sentence, DICO.keys(), n=1, cutoff=0.6)
    if best_match:
        return DICO[best_match[0]]
    return None   # pas de correspondance, on passe à la traduction cloud
```

L’algorithme est invoqué **après** la détection de langue : si `lang === 'wo'` et que le dictionnaire renvoie une traduction, on l’utilise directement ; sinon on délègue à Google ou Azure.  

### 4. Orchestration du flux multilingue  

#### 4.1. Décision « modèle IA »  

| Langue détectée | Modèle recommandé |
|-----------------|-------------------|
| `fr`            | GPT‑4 (ou modèle français spécialisé) |
| `en`            | GPT‑4 (ou modèle anglais) |
| `sw`            | modèle multilingue (ex. **Mistral‑7B‑Instruct** fine‑tuned) |
| `wo`, `ha`      | dictionnaire → fallback cloud (fr) |

Le code suivant montre comment choisir le modèle :

```js
async function generateReply(message, lang) {
  // 1️⃣ Dictionnaire local ?
  if (['wo', 'ha'].includes(lang)) {
    const local = await translateLocal(message, lang);
    if (local) return local;
  }

  // 2️⃣ Modèle IA adapté
  const model = {
    fr: 'openai-gpt4-fr',
    en: 'openai-gpt4-en',
    sw: 'mistral-7b-swahili'
  }[lang] || 'openai-gpt4-fr'; // fallback français

  const response = await callAIModel(message, model);
  return response;
}
```

#### 4.2. Cache des détections  

Les appels aux API de détection coûtent de l’argent et introduisent de la latence. Un **cache en mémoire** (ou Redis) permet de mémoriser les résultats pendant la durée de la session ou pendant 24 h :

```js
const cache = new Map(); // clé = hash(message), valeur = {lang, confidence, ttl}

function getCachedDetection(message) {
  const key = crypto.createHash('md5').update(message).digest('hex');
  const entry = cache.get(key);
  if (entry && Date.now() < entry.expires) return entry;
  return null;
}
```

Le flux devient :  

1. Vérifier le cache → si présent, passer directement au modèle ;  
2. Sinon appeler l’API de détection → stocker le résultat dans le cache ;  
3. Continuer le traitement.  

### 5. Gestion des erreurs et des limites d’API  

| Situation | Action recommandée |
|-----------|-------------------|
| **Erreur réseau / API indisponible** | Retourner le texte brut avec un indicateur « détection en cours… » et réessayer en arrière‑plan. |
| **Quota dépassé** | Basculer sur le dictionnaire local ou sur un modèle open‑source hébergé sur un serveur propre. |
| **Score de confiance bas (< 0,6)** | Proposer à l’utilisateur de préciser la langue : « En quelle langue souhaitez‑vous parler ? » (menu déroulant). |
| **Message trop long (> 5 000 caractères)** | Trancher le texte en fragments, détecter chaque fragment séparément, puis recomposer. |

Ces scénarios sont à tester systématiquement (voir chapitre 7).  

### 6. Enrichissement continu du dictionnaire  

Les langues locales évoluent, de nouveaux termes apparaissent (ex. néologismes liés aux paiements mobiles). Une boucle de **crowdsourcing interne** peut être mise en place :

1. **Capture** : chaque fois qu’une phrase locale n’est pas reconnue, la stocker dans une table `untranslated_phrases`.  
2. **Revue** : un traducteur humain (ou un community manager) valide la traduction.  
3. **Mise à jour** : le dictionnaire JSON est régénéré automatiquement via un script CI/CD.  

```bash
# script de mise à jour (update_dict.sh)
python generate_dict.py untranslated_phrases/ > dictionaries/wo.json
git add dictionaries/wo.json && git commit -m "Update Wolof dictionary"
git push origin main
```

Cette approche garantit que le chatbot reste **pertinent** pour les utilisateurs au fil du temps.  

### 7. Exemple complet : flux d’une conversation en wolof  

1. **Message entrant** : `Nanga def?`  
2. **Détection** : API renvoie `wo` avec 0,92 de confiance.  
3. **Recherche dictionnaire** : correspondance exacte → `Comment ça va ?`  
4. **Envoi au modèle IA** : le texte français est transmis au modèle GPT‑4 français, qui génère la réponse `Je vais bien, merci ! Et vous ?`.  
5. **Traduction retour** : la réponse française est traduite en wolof via le même dictionnaire (ou via le service cloud si aucune entrée).  

```js
// webhook simplifié
app.post('/webhook', async (req, res) => {
  const msg = req.body.message;
  const cached = getCachedDetection(msg);
  const {lang} = cached || await detectLanguage(msg, 'my-gcp-project');
  const local = await translateLocal(msg, lang);
  const userInputFr = local || await translateCloud(msg, lang, 'fr');
  const replyFr = await callAIModel(userInputFr, 'openai-gpt4-fr');
  const replyLocal = await translateLocal(replyFr, lang) || await translateCloud(replyFr, 'fr', lang);
  res.json({reply: replyLocal});
});
```

### 8. Bonnes pratiques de conformité et de confidentialité  

* **Consentement** : informez l’utilisateur que le texte sera transmis à un service tiers (Google, Azure). Un petit bandeau « Vos messages peuvent être analysés pour détecter la langue » suffit (voir chapitre 1).  
* **Masquage des données sensibles** : avant d’appeler l’API de détection, supprimez ou hash les informations personnelles (numéro de téléphone, ID client).  
* **Régions de stockage** : choisissez un datacenter proche de l’Afrique de l’Ouest (ex. `europe-west1`) pour réduire la latence et respecter les exigences locales de souveraineté des données.  

### 9. Tests automatisés de la détection et de la traduction  

```js
// test/detect.test.js
const {expect} = require('chai');
const {detectLanguage} = require('../detection');

describe('Détection de langue', () => {
  it('devrait identifier le swahili', async () => {
    const {lang, confidence} = await detectLanguage('Habari yako?', 'my-gcp-project');
    expect(lang).to.equal('sw');
    expect(confidence).to.be.above(0.85);
  });

  it('devrait fallback sur le français avec un score faible', async () => {
    const {lang, confidence} = await detectLanguage('???', 'my-gcp-project');
    expect(confidence).to.be.below(0.5);
    // le code du flux utilisera le français par défaut
  });
});
```

Intégrer ces tests dans le pipeline CI garantit que les modifications du code ou du dictionnaire ne cassent pas la chaîne de détection.  

## Points clés  

- La détection de langue repose sur des API cloud (Google, Azure) ; choisissez‑les selon le coût, la couverture et la proximité géographique.  
- Un **seuil de confiance** permet de décider quand accepter la détection ou demander une clarification à l’utilisateur.  
- Les langues locales (swahili, wolof, haoussa) bénéficient d’un **dictionnaire personnalisé** ; combinez recherche floue et mise à jour continue via crowdsourcing.  
- Orchestrer le flux : détection → dictionnaire → traduction cloud → modèle IA → traduction retour.  
- Cachez les résultats de détection pour limiter les dépenses et réduire la latence.  
- Prévoyez des mécanismes de repli (fallback) en cas d’erreur d’API, de quota dépassé ou de score de confiance trop bas.  
- Intégrez la conformité (consentement, masquage des données) dès le début du projet.  
- Automatisez les tests de détection et de traduction pour garantir la robustesse du chatbot à chaque itération.