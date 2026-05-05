---
name: day-planner-remote
description: >
  Remote morning planning routine. Reads Google Calendar and Gmail,
  produces a prioritized daily plan, and creates tasks in Notion and Google Calendar.
  Designed to run automatically every weekday morning at 08:00 Paris time.
  Does NOT perform Gmail actions (labels/archive) — handled by local routine.
compatibility:
  tools:
    - Gmail (search_threads — read only)
    - Google Calendar (list_events, create_event)
    - Notion (notion-create-pages, notion-search)
context:
  timezone: Europe/Paris
  language: French
  work_days: Monday to Friday
---

# Day Planner — Remote Routine (Planning)

Routine matinale cloud. Lit le calendrier et les mails, génère un plan
priorisé, et crée les tâches dans Notion et Google Calendar.
Le tri de la boîte mail est géré séparément par la Local Routine.

---

## Workflow

### 1. Date et contexte

Récupérer la date du jour (timezone: Europe/Paris).
Le plan couvre **aujourd'hui uniquement**.

---

### 2. Calendrier du jour

Utiliser **Google Calendar: list_events** :
- `timeMin` = 00:00 heure locale
- `timeMax` = 23:59 heure locale
- Pour chaque événement noter : titre, heure, durée, participants, lien de réunion

---

### 3. Emails — Lecture et analyse

Utiliser **Gmail: search_threads** :
```
is:unread OR is:important newer_than:2d
```
Récupérer jusqu'à 20 threads. Pour chaque thread noter : expéditeur, sujet, date, snippet.

Classifier chaque mail :
| Catégorie | Critères |
|-----------|----------|
| 🔴 Urgent | Action requise aujourd'hui (deadline, réponse attendue, bloque quelqu'un) |
| 🟡 Important | À traiter cette semaine, pas forcément aujourd'hui |
| ⚪ Low priority | FYI, newsletters, notifications automatiques, promos |

> Note : le tri, archivage et labellisation sont effectués par la Local Routine (gmail-mcp).

---

### 4. Plan du jour

Générer le plan dans ce format :

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
```

**Logique de priorisation :**
1. Meetings imminents / deadlines du jour
2. Tâches qui bloquent d'autres personnes
3. Réponses dues (mails urgents)
4. Deep work (recherche d'emploi, développement, freelance)
5. Tâches administratives

---

### 5. Créer les tâches dans Notion

Utiliser **Notion: notion-search** pour trouver la base de données de tâches
(chercher : "tâches", "tasks", "todo", "planning").

Pour chaque tâche prioritaire (🔴 et 🟡), créer une page avec **Notion: notion-create-pages** :
- Titre : nom de la tâche
- Date : aujourd'hui
- Priorité : High / Medium selon l'emoji
- Notes : contexte source (sujet du mail ou nom du meeting)

---

### 6. Créer les blocs dans Google Calendar

Utiliser **Google Calendar: create_event** pour bloquer les créneaux disponibles :

**Heuristique de placement :**
- 9h–12h → Deep work (blocs 60–90 min)
- 12h–14h → Pause / libre
- 14h–16h → Meetings, collaboration
- 16h–18h → Admin, réponses rapides (blocs 15–30 min)

Pour chaque événement :
- Titre : nom de la tâche
- Description : source (sujet mail / meeting associé)
- Rappel : 10 minutes avant

---

### 7. Résumé final

```
✅ Planning du jour prêt !
→ X tâches créées dans Notion
→ X blocs ajoutés au calendrier

💡 Lance la Local Routine sur ton Mac pour trier ta boîte mail.

Bonne journée 💪
```

---

## Règles générales

- **Langue** : toujours répondre en français
- **Ton** : concis, actionnable
- **Autonomie** : ne pas poser de questions, prendre des décisions raisonnables
- **Erreurs** : si un outil échoue, continuer et noter l'erreur dans le résumé

## Edge cases

- **Aucun mail non lu** : noter "Boîte mail à jour ✅"
- **Calendrier plein** : ne pas créer de blocs Calendar, suggérer de reporter
- **Aucune base Notion trouvée** : créer dans le workspace par défaut
