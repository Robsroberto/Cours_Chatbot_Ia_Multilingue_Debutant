## Fondamentaux du traitement du langage naturel  

Le traitement du langage naturel (NLP) repose sur trois opérations de base qui transforment un texte brut en une forme exploitable par une intelligence artificielle :  

| Étape | Objectif | Exemple concret (client d’une boutique en ligne à Abidjan) |
|-------|----------|------------------------------------------------------------|
| **Tokenisation** | Découper le texte en unités (mots, sous‑mots, ponctuation). | `"Je veux changer ma taille"` → `["Je","veux","changer","ma","taille"]` |
| **Normalisation** | Uniformiser les formes (minuscules, suppression des accents, lemmatisation). | `"Télècharge"` → `"telecharger"` |
| **Vectorisation** | Convertir chaque token en un vecteur numérique (embedding) qui capture le sens. | Le mot *taille* obtient un vecteur de 768 dimensions qui le place près de *dimension*, *mesure*, etc. |

Ces étapes permettent à un modèle de « comprendre » le texte sans le lire comme un humain. En pratique, les plateformes no‑code intègrent déjà ces pré‑traitements ; il suffit de fournir le texte brut et le service s’occupe du reste.

### Tokenisation et sous‑mots  

Dans les langues africaines, les espaces ne séparent pas toujours les morphèmes (ex. : *kpakpa* en lingala). Les tokeniseurs basés sur les **Byte‑Pair Encoding (BPE)** ou **WordPiece** découpent les mots rares en sous‑unités fréquentes, ce qui améliore la couverture lexicale.  

> *Cas pratique* : un client en langue bambara écrit « sagaba » (bonjour). Le tokeniseur BPE le découpe en `["sa","ga","ba"]`, chaque sous‑mot étant déjà connu du modèle, évitant ainsi un *unknown token*.

### Embeddings multilingues  

Les **embeddings** sont des vecteurs qui représentent le sens d’un token. Les modèles multilingues (ex. : mBERT) apprennent un espace partagé où le même concept possède une représentation proche, quel que soit le langage : le vecteur de *order* (anglais) sera proche de celui de *commande* (français) ou *commande* (wolof). Cette propriété est cruciale pour un chatbot qui doit répondre dans plusieurs langues sans entraîner un modèle séparé pour chaque langue.

---

## Deux grandes tâches pour les chatbots  

Un chatbot IA doit généralement accomplir deux fonctions distinctes : identifier l’intention de l’utilisateur et produire une réponse adaptée.  

### Classification d’intention  

Il s’agit d’attribuer une catégorie pré‑définie à chaque message (ex. : *demande de suivi de commande*, *réclamation*, *prise de rendez‑vous*). La plupart des plateformes no‑code offrent un **bloc d’intention** où l’on fournit des exemples d’utterances ; le moteur sous‑jacent entraîne un classifieur (souvent un petit réseau de type **logistic regression** ou **tiny BERT**) en arrière‑plan.  

*Illustration* :  

```json
{
  "intents": [
    {
      "name": "suivi_commande",
      "examples": [
        "Où est ma commande ?",
        "Quel est le statut de mon colis ?",
        "Suivi de livraison"
      ]
    },
    {
      "name": "reclamation",
      "examples": [
        "Le produit est cassé",
        "Je veux me faire rembourser",
        "Problème de facturation"
      ]
    }
  ]
}
```

Le classifieur compare le vecteur du message entrant à ceux des exemples et renvoie le label le plus probable.  

### Génération de texte  

Une fois l’intention identifiée, le bot doit formuler une réponse. Deux approches sont possibles :  

1. **Réponses pré‑définies** (templates). Simple à gérer, mais peu flexible.  
2. **Génération dynamique** à l’aide d’un modèle de type **GPT** ou **T5**. Le modèle reçoit le contexte (intention, entités extraites, langue) et produit un texte naturel.  

Dans un contexte africain où les clients utilisent souvent un mélange de français, anglais et langues locales, la génération dynamique permet d’ajuster le ton, les formules de politesse et les références culturelles sans devoir écrire des centaines de variantes manuellement.

---

## Multilinguisme : enjeux et solutions  

### Variabilité linguistique en Afrique  

Le continent regroupe plus de 2000 langues ; même dans les zones francophones, le **français** coexiste avec des langues locales (swahili, haoussa, yoruba, zoulou, etc.) et des créoles. Les défis majeurs sont :  

* **Orthographe non standardisée** : les utilisateurs écrivent souvent « salam », « salaam », ou « salaam » pour dire *bonjour* en arabe.  
* **Code‑switching** : un même message peut contenir plusieurs langues (« Je veux le produit, mais can you ship it to Lagos ? »).  
* **Ressources limitées** : peu de corpus annotés pour l’entraînement de modèles spécifiques.  

### Approches de traduction vs modèles multilingues  

| Approche | Avantages | Inconvénients |
|----------|-----------|---------------|
| **Traduction externe** (Google Cloud Translation, DeepL) | Couverture quasi‑universelle, pas besoin de former de modèle. | Latence supplémentaire, coût à l’usage, perte de nuances culturelles. |
| **Modèles multilingues (mBERT, XLM‑R, mT5)** | Traitement en‑ligne, même vecteur pour le même concept quelle que soit la langue, possibilité de fine‑tuning sur données locales. | Nécessite un service d’inférence (ex. : Hugging Face Inference API) et parfois plus de puissance de calcul. |

Dans un projet no‑code, il est souvent judicieux de combiner les deux : détecter la langue, puis, si le modèle multilingue ne supporte pas la langue cible, recourir à la traduction. Cette stratégie garantit une réponse rapide pour les langues majeures (français, anglais) tout en restant ouvert aux langues moins courantes.

---

## Modèles pré‑entraînés accessibles sans code  

### GPT et ses dérivés  

Les modèles **GPT‑3** et **GPT‑4** d’OpenAI offrent une API REST qui accepte simplement une chaîne de texte et renvoie une continuation. Les constructeurs de chatbot (ex. : Landbot, Voiceflow) intègrent déjà un **connector** : il suffit de renseigner la clé API, le prompt et le nombre de tokens souhaités.  

*Exemple de prompt* :  

```
You are a friendly customer support agent for an e‑commerce store in Côte d'Ivoire. Answer in French, but switch to the language the user used if you detect it.
User: Où est ma commande ?
```

Le modèle renvoie :  

```
Bonjour ! Votre commande n°12345 est en cours de livraison et devrait arriver d'ici demain. Vous pouvez suivre le colis ici : https://... 
```

### BERT et mBERT  

**BERT** (Bidirectional Encoder Representations from Transformers) est surtout utilisé pour la classification (intention, sentiment). **mBERT** (multilingual BERT) a été entraîné sur 104 langues, dont le français, l'anglais, le swahili et le haoussa. Sur les plateformes no‑code, le bloc *Intent Recognizer* repose souvent sur une version allégée de BERT ; l’utilisateur ne voit jamais le code, mais il peut choisir le modèle « multilingual » dans les paramètres.  

### Utilisation via plateformes no‑code  

| Plateforme | Fonctionnalité multilingue native | Exemple d’intégration |
|------------|-----------------------------------|------------------------|
| **Chatfuel** | Bloc *AI Setup* avec support de plusieurs langues. | Dans le bloc, sélectionner « Multilingual », ajouter les intents en français et en anglais. |
| **ManyChat** | Action *Detect Language* + *Google Translate* intégré. | Ajouter une action « Detect Language », puis un *Condition* : si `lang = "fr"` → réponse française, sinon → passer à la traduction. |
| **Landbot** | Connecteur *OpenAI* + *Google Cloud Translation* en chaîne. | Créer un flux : `User Input → Detect Language → (if not supported) → Translate → GPT → Translate back`. |
| **Voiceflow** | Bloc *NLU* avec modèle pré‑entraîné multilingue. | Configurer les intents en plusieurs langues dans le même bloc NLU. |

Ces solutions permettent de **déployer un chatbot multilingue sans écrire une seule ligne de code**, tout en gardant la possibilité de personnaliser le prompt ou les exemples d’intents pour mieux coller aux réalités locales (ex. : termes de paiement mobile comme *MTN Mobile Money* ou *Orange Money*).

---

## Choisir la langue de réponse en temps réel  

### Détection automatique de la langue  

La première étape consiste à identifier la langue du message entrant. Les services les plus courants sont :  

* **Google Cloud Translation – DetectLanguage**  
* **Microsoft Azure – Text Analytics**  
* **Langdetect (Python)** – utilisable via des fonctions serverless si la plateforme le permet.  

Exemple d’appel API (REST) :  

```http
POST https://translation.googleapis.com/language/translate/v2/detect
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "q": "Mbaadi, comment puis-je payer avec Orange Money ?"
}
```

Réponse :  

```json
{
  "data": {
    "detections": [
      [
        {
          "language": "fr",
          "confidence": 0.98,
          "isReliable": true
        }
      ]
    ]
  }
}
```

Le champ `language` indique la langue détectée (`fr` pour français). Sur les plateformes no‑code, ce résultat est souvent stocké dans une variable (`{{detected_lang}}`) que l’on utilise dans les conditions de flux.

### Routage vers le modèle ou la traduction appropriée  

Une fois la langue connue, le bot décide :  

1. **Langue supportée par le modèle multilingue** → appeler directement GPT/mBERT avec le prompt dans la langue détectée.  
2. **Langue non supportée** → traduire le texte en français (ou en anglais), générer la réponse, puis retraduire dans la langue d’origine.  

*Flux typique* :  

```
User Input → Detect Language → 
   ├─ (langue supportée) → GPT (prompt in detected language) → Send response
   └─ (langue non supportée) → Translate to FR → GPT (FR prompt) → Translate back → Send response
```

Ce schéma minimise la latence pour les langues courantes tout en garantissant une couverture maximale.

---

## Bonnes pratiques pour un chatbot multilingue robuste  

| Pratique | Pourquoi c’est important | Mise en œuvre concrète |
|----------|--------------------------|------------------------|
| **Enrichir les exemples d’intents dans chaque langue** | Améliore la précision du classifieur, surtout avec le code‑switching. | Ajouter des utterances mixtes (« Je veux payer avec M‑Money », « Can I use Mobile Money ? ») dans le bloc d’intention. |
| **Normaliser les variantes orthographiques** | Réduit le nombre d’utterances uniques à gérer. | Créer une fonction de pré‑traitement qui remplace « salaam », « salam », « salām » par un token commun `SALAM`. |
| **Surveiller la confiance de la détection de langue** | Évite d’envoyer une réponse dans la mauvaise langue. | Si `confidence < 0.80`, demander à l’utilisateur « En quelle langue préférez‑vous continuer ? ». |
| **Limiter le nombre de tours de traduction** | Chaque appel API a un coût et augmente la latence. | Prioriser les langues les plus fréquentes (fr, en, sw) et n’utiliser la traduction qu’en dernier recours. |
| **Tester avec des locuteurs natifs** | Les modèles peuvent générer des tournures inappropriées. | Organiser des sessions de test avec des utilisateurs de chaque région cible (ex. : Dakar, Bamako, Kinshasa). |
| **Mettre à jour régulièrement les intents** | Les besoins évoluent (nouveaux services, promotions). | Exporter les logs, identifier les nouvelles utterances non reconnues, les ajouter au fichier d’intents. |

---

## Points à retenir  

- Le NLP transforme le texte brut en vecteurs grâce à la tokenisation, la normalisation et les embeddings ; les modèles multilingues partagent un même espace sémantique pour toutes les langues.  
- Un chatbot combine **classification d’intention** (déterminer le besoin) et **génération de texte** (produire la réponse) ; les deux tâches peuvent être réalisées sans écrire de code grâce aux blocs NLU et aux connecteurs IA des plateformes no‑code.  
- Le multilinguisme en Afrique implique de gérer l’orthographe variable, le code‑switching et la rareté des ressources ; les stratégies hybrides (modèles multilingues + traduction) offrent le meilleur compromis entre couverture, coût et latence.  
- Les modèles pré‑entraînés (GPT, BERT, mBERT) sont accessibles via des API ou des connecteurs intégrés ; choisir la version **