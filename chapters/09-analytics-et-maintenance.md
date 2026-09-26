## Exploiter les tableaux de bord pour piloter votre chatbot

Les plateformes no‑code offrent aujourd’hui des interfaces visuelles où chaque échange est consigné : texte, langue, durée, score de confiance du modèle, etc. Ces données sont accessibles via des **tableaux de bord** (dashboards) qui permettent de :

* Visualiser le volume d’interactions par canal (site web, WhatsApp, Facebook Messenger).  
* Identifier les langues les plus fréquentes et les variations dialectales (p.ex. français vs. français congolais).  
* Mesurer les indicateurs de performance (taux de résolution, temps moyen de réponse, satisfaction client).  
* Découvrir les **intentions non reconnues** (messages qui n’ont pas déclenché d’intent ou qui ont reçu le fallback “Je ne comprends pas”).

### Configurer le tableau de bord de base

1. **Sélection des métriques**  
   - **Sessions** : nombre d’utilisateurs distincts.  
   - **Messages** : total de messages échangés.  
   - **Intentions reconnues** vs. **fallbacks**.  
   - **Langue détectée** (exemple : `fr`, `en`, `sw`).  
   - **Score de confiance** du classificateur (0 – 1).  

2. **Création de filtres**  
   - Par période (jour, semaine, mois).  
   - Par canal (WhatsApp, site web).  
   - Par segment d’utilisateur (nouveaux vs. récurrents).  

3. **Alertes automatisées**  
   La plupart des solutions (Landbot, ManyChat) permettent de définir des seuils :  
   ```json
   {
     "metric": "fallback_rate",
     "threshold": 0.12,
     "period": "daily",
     "action": "send_email",
     "recipients": ["admin@entreprise.af"]
   }
   ```
   Quand le taux de fallback dépasse 12 % en une journée, un mail d’alerte est envoyé.  

> **Astuce** : activez les alertes dès le lancement du bot pour réagir avant que les utilisateurs ne se découragent.

---

## Détecter les nouvelles intentions sans coder

### Analyse des logs de fallback

Chaque fois que le bot ne trouve pas d’intent, le message est stocké dans les logs. Exportez‑les régulièrement (CSV ou via API) et appliquez une **analyse de fréquence** :

```python
import pandas as pd
from collections import Counter
import re

df = pd.read_csv('fallback_logs.csv')
messages = df['user_message'].dropna().tolist()

# Nettoyage basique
def clean(txt):
    txt = txt.lower()
    txt = re.sub(r'[^a-zàâçéèêëîïôûùüÿñæœ\s]', '', txt)
    return txt.strip()

cleaned = [clean(m) for m in messages]
counter = Counter(cleaned)

# Affiche les 10 messages les plus fréquents
for phrase, freq in counter.most_common(10):
    print(f"{phrase}: {freq}")
```

Les phrases récurrentes indiquent souvent des **intentions manquantes** (ex. : “Comment suivre ma commande ?” alors que le bot ne possède que “Suivi de livraison”).  

### Créer un intent à la volée (no‑code)

Sur la plupart des plateformes, il suffit de :

1. Ouvrir le **module Intent Management**.  
2. Cliquer sur **“Nouvel intent”**.  
3. Copier‑coller les exemples de phrases extraites du script précédent (au moins 5 variantes).  
4. Assigner une réponse ou un **flow** existant (redirection vers un agent humain, affichage d’une FAQ, etc.).  

Aucun script n’est requis ; la mise à jour est instantanée et le bot commence à reconnaître l’intent dès la prochaine session.

> **Voir chapitre 6** pour la logique de création d’intents dans le modèle IA.  

---

## Calendrier de maintenance : quoi faire, quand, et pourquoi

| Période | Action | Objectif |
|---------|--------|----------|
| **Quotidien** | Vérifier les alertes de fallback et les pics de latence. | Réagir rapidement à un problème de reconnaissance ou de performance. |
| **Hebdomadaire** | Exporter les logs de fallback, analyser les nouvelles expressions, créer ou enrichir les intents. | Améliorer continuellement la couverture linguistique. |
| **Mensuel** | Re‑entraîner le modèle avec les nouvelles données (FAQ mises à jour, nouveaux produits). | Garantir que le modèle reste aligné sur l’offre commerciale. |
| **Trimestriel** | Auditer les coûts d’infrastructure (API de traduction, appels OpenAI). Négocier les forfaits ou changer de fournisseur si nécessaire. | Optimiser le budget et éviter les surprises de facturation. |
| **Annuel** | Revoir la stratégie multilingue : ajouter de nouvelles langues (p.ex. : lingala, haoussa) ou désactiver celles qui ne sont plus pertinentes. | Adapter le bot à l’évolution du marché. |

### Outils de planification

- **Google Calendar** partagé avec l’équipe support.  
- **Zapier** ou **Make (Integromat)** pour déclencher automatiquement des tâches (ex. : envoyer un rappel Slack chaque lundi pour “Analyser les fallbacks”).  

```yaml
# Exemple de scénario Make
trigger:
  - schedule: every monday 09:00
actions:
  - google_sheets.append:
      file_id: "1AbcDeF..."
      sheet_name: "Fallback_Review"
      rows: "{{fallback_logs}}"
  - slack.send_message:
      channel: "#support-bot"
      text: "✅ Rapport hebdo des fallbacks ajouté à Sheets."
```

---

## Stratégies de formation continue du modèle IA

### 1. **Apprentissage incrémental (fine‑tuning) périodique**

- **Collecte** : regroupez les conversations validées (intents correctement détectés) et les cas d’échec corrigés.  
- **Étiquetage** : utilisez les outils de labellisation intégrés (ex. : “Mark as intent X”).  
- **Fine‑tuning** : sur les plateformes comme OpenAI, déclenchez un job de fine‑tuning avec le nouveau dataset :

```bash
openai api fine_tunes.create \
  -t new_dataset.jsonl \
  -m ada \
  --suffix "v2_2024_09"
```

Planifiez ce job **une fois par mois** pour limiter le coût tout en intégrant les nouvelles expressions.

### 2. **Enrichissement de la base de connaissances**

- **FAQ dynamique** : ajoutez les réponses qui ont reçu un score de satisfaction > 4/5 (sur 5).  
- **Glossaire de termes locaux** : créez un dictionnaire de mots spécifiques à chaque pays (ex. : “M-Pesa” en Kenya, “Boulot” en Côte d’Ivoire).  
- **Mise à jour automatisée** : via un webhook qui, à chaque nouvelle entrée validée, pousse la phrase dans le dataset d’entraînement.

```json
{
  "event": "intent_added",
  "payload": {
    "intent": "paiement_mpesaplus",
    "examples": [
      "Comment payer avec M‑Pesa ?",
      "Puis‑je utiliser M‑Pesa pour ma facture ?"
    ]
  }
}
```

### 3. **Évaluation continue (A/B testing)**

Créez deux versions du même intent : une **baseline** (modèle actuel) et une **version améliorée** (avec nouvelles données). Divisez le trafic 50/50 et comparez :

- **Taux de résolution**  
- **Score de confiance**  
- **Temps de réponse**  

Conservez la version qui obtient les meilleures métriques. Cette approche minimise les risques liés à un déploiement massif d’un modèle non testé.

---

## Réduire les coûts d’infrastructure sans sacrifier la qualité

### Optimiser les appels d’API de traduction

- **Cache local** : stockez les traductions fréquentes (ex. : réponses standards) dans une base clé‑valeur (Redis, ou même un simple fichier JSON).  
- **Batching** : regroupez plusieurs phrases avant d’appeler l’API ; la plupart des services offrent un tarif dégressif par lot.

```python
import redis, json, time
r = redis.Redis(host='localhost', port=6379, db=0)

def translate_batch(texts, target='fr'):
    cache_key = f"trans:{hash(tuple(texts))}:{target}"
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)

    # appel API (exemple fictif)
    translations = external_translation_service(texts, target)
    r.setex(cache_key, 86400, json.dumps(translations))  # 24 h d’expiration
    return translations
```

### Limiter les appels au modèle de génération

- **Réponses pré‑générées** pour les questions les plus courantes (ex. : horaires d’ouverture, politique de retour).  
- **Déclenchement conditionnel** : n’appeler le modèle que lorsqu’aucune réponse fixe ne correspond, en utilisant un **router** logique.

```yaml
# Exemple de flux Landbot
- condition: "{{intent}} == 'fallback'"
  actions:
    - call_openai: "{{user_message}}"
- else:
  - send_message: "{{predefined_answer}}"
```

### Choisir le bon plan tarifaire

- **Analyse du volume** : utilisez les métriques du tableau de bord pour projeter le nombre d’appels mensuels.  
- **Plan à la consommation vs. forfait** : si le trafic est prévisible (ex. : boutique en ligne avec 200 k visites/mois), un forfait fixe peut réduire les dépenses.  
- **Partenariats locaux** : certains fournisseurs cloud (Google Cloud Africa, Azure Afrique) proposent des tarifs préférentiels pour les startups africaines.

---

## Mise en place d’un processus de gouvernance

1. **Rôle du “Data Steward”**  
   - Responsable de la qualité des logs, de l’étiquetage et de la conformité (RGPD, loi sur la protection des données en Afrique).  

2. **Comité de pilotage mensuel**  
   - Revue des KPI (taux de résolution, coût par interaction).  
   - Décisions sur l’ajout d’intents ou la mise à jour du modèle.  

3. **Documentation vivante**  
   - Chaque modification d’intent ou de flow doit être consignée dans un **wiki** (Confluence, Notion) avec la date, le responsable et le motif.  

Cette gouvernance garantit que le bot évolue de façon **contrôlée**, évitant les dérives fonctionnelles et les dépassements budgétaires.

---

## Points clés

- **Tableaux de bord** : centralisent volume, langues, taux de fallback ; configurez des alertes pour réagir rapidement.  
- **Détection d’intents manquants** : analysez les logs de fallback, créez de nouveaux intents en quelques clics, sans toucher au code.  
- **Calendrier de maintenance** : tâches quotidiennes (alertes), hebdomadaires (analyse des fallbacks), mensuelles (fine‑tuning), trimestrielles (audit des coûts) et annuelles (révision multilingue).  
- **Formation continue** : fine‑tuning mensuel, enrichissement de la FAQ, A/B testing pour valider les améliorations.  
- **Optimisation des coûts** : mise en cache des traductions, réponses pré‑générées, routage conditionnel, choix du forfait adapté.  
- **Gouvernance** : rôle dédié au contrôle des données, comité de pilotage mensuel et documentation dynamique pour assurer la conformité et la traçabilité.  

En appliquant ces pratiques, votre chatbot IA restera pertinent, performant et économique, même à mesure que votre activité s’étend à de nouveaux marchés francophones et multilingues en Afrique.