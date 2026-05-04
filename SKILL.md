---
name: day-planner
description: >
  Morning daily planning routine for Claude Code. Reads Google Calendar and Gmail,
  categorizes and archives emails via gmail-mcp, produces a prioritized daily plan,
  and creates tasks in Notion and/or Google Calendar.
  Designed to run automatically every weekday morning at 08:00 Paris time.
compatibility:
  tools:
    - gmail-mcp (create_label, label_thread, archive_thread, search_threads)
    - Google Calendar (list_events, create_event)
    - Notion (notion-create-pages, notion-search)
context:
  user: Kevin
  timezone: Europe/Paris
  language: French
  work_days: Monday to Friday
---

# Day Planner — Routine Claude Code

Routine matinale autonome. S'exécute sans interaction utilisateur.
Lit le calendrier et les mails, trie la boîte, génère un plan priorisé,
et crée les tâches dans Notion et Google Calendar.

---

## Workflow complet

### 1. Date et contexte

Récupérer la date du jour (timezone: Europe/Paris).
Le plan couvre **aujourd'hui uniquement**.

---

### 2. Calendrier du jour

Utiliser **Google Calendar: list_events** :
- `timeMin` = 00:00 heure locale
- `timeMax` = 23:59 heure locale
- Pour chaque événement, noter : titre, heure, durée, participants, lien de réunion

---

### 3. Emails — Lecture et tri

#### 3a. Récupérer les mails

Utiliser **gmail-mcp: search_threads** :
```
is:unread OR is:important newer_than:2d
```
Récupérer jusqu'à 20 threads. Pour chaque thread, noter : expéditeur, sujet, date, snippet.

#### 3b. Classifier chaque mail

| Catégorie | Critères |
|-----------|----------|
| 🔴 Urgent | Action requise aujourd'hui (deadline, réponse attendue, bloque quelqu'un) |
| 🟡 Important | À traiter cette semaine, pas forcément aujourd'hui |
| ⚪ Low priority | FYI, newsletters, notifications automatiques, promos |

#### 3c. Trier la boîte avec gmail-mcp

**Pour les mails ⚪ Low priority :**
- Utiliser **gmail-mcp: archive_thread** pour les archiver

**Pour les mails 🔴 et 🟡, appliquer les labels suivants :**

| Type de mail | Label |
|-------------|-------|
| Offres d'emploi, alertes LinkedIn | `Job Search` |
| Factures, tickets d'achat | `Factures` |
| Relevés, fiscalité, investissements | `Finances` |
| Supabase, GitHub, dev tools | `Dev` |
| Abonnements, services en ligne | `Admin` |
| Commandes, livraisons e-commerce | `Achats` |
| Sécurité, connexions suspectes | `Sécurité` |
| Course à pied, vélo, sport | `Sport` |
| Personnel, voisinage, famille | `Perso` |
| Freelance, clients, prospects | `Freelance` |

Avant d'appliquer un label, vérifier s'il existe avec **gmail-mcp: list_labels**.
Si le label n'existe pas, le créer avec **gmail-mcp: create_label**, puis l'appliquer.

---

### 4. Plan du jour

Générer le plan dans ce format exact :

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📅 PLAN DU JOUR — [Jour] [Date]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⏰ AGENDA
[Événements avec horaires, ou "Aucun meeting aujourd'hui — journée libre 🎯"]

🎯 PRIORITÉS DU JOUR
1. 🔴 [Tâche] — [source] (~Xmin)
2. 🟡 [Tâche] — [source] (~Xmin)
3. ...

📬 MAILS URGENTS
[Uniquement les 🔴, avec action recommandée]
[Ou "Boîte mail à jour ✅" si aucun urgent]

📊 BOÎTE MAIL
✅ X mails archivés
🏷 X mails catégorisés
```

**Logique de priorisation des tâches :**
1. Meetings imminents / deadlines du jour
2. Tâches qui bloquent d'autres personnes
3. Réponses dues (mails urgents)
4. Deep work (recherche d'emploi, développement, freelance)
5. Tâches administratives

---

### 5. Créer les tâches dans Notion

Utiliser **Notion: notion-search** pour trouver la base de données de tâches
(chercher : "tâches", "tasks", "todo", "planning", "aujourd'hui").

Pour chaque tâche prioritaire (🔴 et 🟡), créer une page avec **Notion: notion-create-pages** :
- Titre : nom de la tâche
- Date : aujourd'hui
- Priorité : High / Medium selon l'emoji
- Notes : contexte source (sujet du mail ou nom du meeting)

Si aucune base trouvée : créer les pages dans le workspace par défaut et le mentionner.

---

### 6. Créer les blocs dans Google Calendar

Utiliser **Google Calendar: create_event** pour bloquer les créneaux disponibles :

**Heuristique de placement :**
- 9h–12h → Deep work (blocs 60–90 min)
- 12h–14h → Pause / libre
- 14h–16h → Meetings, collaboration
- 16h–18h → Admin, réponses rapides (blocs 15–30 min)

Pour chaque événement créé :
- Titre : nom de la tâche
- Description : source (sujet mail / meeting associé)
- Rappel : 10 minutes avant

Ne pas créer de blocs si le calendrier est déjà plein sur ce créneau.

---

### 7. Résumé final

Terminer avec un message de confirmation concis :

```
✅ Planning du jour prêt !
→ X tâches créées dans Notion
→ X blocs ajoutés au calendrier
→ Boîte mail triée (X archivés, X catégorisés)

Bonne journée Kevin 💪
```

---

## Règles générales

- **Langue** : toujours répondre en français
- **Ton** : concis, actionnable, pas de blabla
- **Autonomie** : cette routine tourne sans interaction — ne pas poser de questions, prendre des décisions raisonnables
- **Erreurs** : si un outil échoue, continuer avec les autres et noter l'erreur dans le résumé final

## Edge cases

- **Aucun mail non lu** : skip section mail, noter "Boîte mail à jour ✅"
- **Calendrier plein** : ne pas créer de blocs Calendar, suggérer de reporter les tâches basses priorité
- **Aucune base Notion trouvée** : créer dans le workspace par défaut
- **Jour férié / week-end** : la routine ne doit pas tourner (configurer le schedule lun-ven uniquement)
