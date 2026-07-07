# Les cours de Lou — Planning Pilates

Page web unique qui rassemble en un seul planning tous les cours de **Lou Carretero**,
instructrice de Pilates, à partir des deux studios où elle enseigne :

- **El Pilates Studio** (Cranves-Sales, France) — réservations via [bsport](https://backoffice.bsport.io/m/El%20pilates%20Studio/5489/calendar?tabSelected=0)
- **Champel Santé** (Genève, Suisse) — réservations via [champelsante.ch/planning](https://www.champelsante.ch/planning)

Pour chaque cours, la page affiche : la date, l'heure, la durée, le type de cours,
le lieu, le prix, l'état (complet ou non) et un bouton **Réserver** :

- **El Pilates Studio** : lien vers le calendrier bsport du studio, positionné
  sur la date du cours et filtré sur les cours de Lou
  (`…/calendar?tabSelected=0&date=<AAAA-MM-JJ>&coaches=128693`). Le détail d'un
  cours s'ouvre en pop-up sur ce calendrier et n'a pas d'URL propre.
- **Champel Santé** : lien vers la page planning du site — leur réservation
  s'ouvre dans une fenêtre sur cette page, il n'existe ni URL par séance ni
  paramètre de date (vérifié : la page ignore `?date=…`).

## Fonctionnement

Le site est une **page statique unique** (`index.html`), sans serveur ni build :
le navigateur interroge directement les APIs publiques des deux studios au chargement.

| Source | API | Filtre |
|---|---|---|
| El Pilates Studio | `api.production.bsport.io/api/v1/offer/minimal/` (+ `meta-activity`, `establishment`) | `company=5489`, `coach=128693` (Lou), cours futurs uniquement, remplacements inclus via `coach_override` |
| Champel Santé | Supabase REST (`knucrtezvcxsdxrwgzdq.supabase.co/rest/v1/sessions`) avec la clé publique du site | `instructor_id` de Lou, statut `confirmed`, dates futures |

Les deux APIs autorisent les requêtes cross-origin (CORS), la page fonctionne donc
depuis n'importe quel hébergement statique. Si une source est indisponible, l'autre
s'affiche quand même avec un avertissement.

**Cas particulier** : les noms des cours de Champel Santé viennent de leur CMS
(Sanity), qui bloque les requêtes navigateur tierces. La correspondance
`id → nom du cours` est donc embarquée dans `index.html`
(constante `CHAMPEL_ACTIVITIES`). Si Champel Santé ajoute un nouveau type de cours,
il apparaîtra comme « Cours » tant que la table n'est pas complétée.

## Hébergement (GitHub Pages)

1. Dans le dépôt GitHub : **Settings → Pages**
2. Source : **Deploy from a branch**, choisir la branche et le dossier `/ (root)`
3. La page sera disponible sur `https://<compte>.github.io/PlanningPilates/`

Aucune étape de build n'est nécessaire.

## Maintenance

- **Horizon d'affichage** : constante `HORIZON_DAYS` dans `index.html` (42 jours par défaut).
  El Pilates Studio publie son planning environ 2 semaines à l'avance.
- **Identifiants** : si Lou change de compte chez un studio, mettre à jour
  `BSPORT.coachId` ou `CHAMPEL.instructorId` dans `index.html`.
- **Nouveau studio** : ajouter une fonction `fetch...()` qui renvoie des objets
  `{ studio, start, end, title, place, full, price, bookUrl }` et l'ajouter au
  `Promise.allSettled` final.
