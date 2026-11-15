# CONTEXTE — Atlas d'anatomie 3D

À lire en premier par toute IA qui continue ce projet. Le propriétaire est un étudiant en 1ʳᵉ année de médecine (France) ; l'interface et les noms sont en français. Réponses courtes, une tâche à la fois.

## Objectif
Application web d'anatomie 3D complète, fluide et belle, du niveau de Complete Anatomy, Visible Body ou Anatomy Learning. Modèles : Z-Anatomy (CC BY-SA 4.0). Usage visé : Mac et téléphone (installable, fonctionne hors-ligne).
Priorités : fluidité, quiz d'entraînement, os et reliefs osseux précis, ergonomie.

## Dépôts
- `mathmar27/atlas` : **dépôt de travail**. Toutes les mises à jour se font ici.
- `mathmar27/atlas-anatomie` : figé (version d'avant le mode quiz « Écrire le nom »). Ne pas y toucher.
- Publication : GitHub Pages → https://mathmar27.github.io/atlas/ (fichier `.nojekyll` présent).

## Organisation des fichiers
- `index.html` : structure de la page, tout le CSS, les panneaux. Charge `app.js?v=N`.
- `app.js` : **tout le code** (~1400 lignes très denses, JavaScript pur, pas de build ni de bundler).
- `export/*.glb` : modèles 3D par système (squelette, articulations, muscles, vaisseaux, nerfs, viscères, fascias, bourses…) + `reperes_os.json`.
- `data/catalog.json` : noms français/latins/anglais, reliefs osseux (`landmarks`). `data/muscles.json` : fiches muscles (origine, terminaison, innervation, action).
- `sw.js` : service worker (cache hors-ligne, nom `atlas-vN`). `manifest.webmanifest` + icônes : installation sur téléphone.
- Bibliothèques : three.js 0.160 (importmap, jsdelivr), GLTFLoader + Draco, OrbitControls, three-mesh-bvh (chargé en option).

## Lancer et tester
- En local : `python3 -m http.server 8000` à la racine du dépôt, puis http://localhost:8000/ .
- Le dossier `export` est détecté automatiquement (`export/` ou `../export/`).
- Console du navigateur : `window.__atlas` expose `renderer, scene, parts, SYS, SYSID, S, LM, Q, PQ, TY, NAV, CAT, camera, controls, select, isolate, openLm, setSys, applyView, VIEWS, quizSets, tyStart, pqStart, navOpen, invalidate…` — pratique pour tester sans cliquer.
- **Après chaque modification** : incrémenter `app.js?v=N` dans `index.html` ET le nom du cache dans `sw.js` (`atlas-vN`), sinon le navigateur garde l'ancienne version. En test : désinscrire le service worker et vider les caches (ou Cmd+Maj+R).
- Vérifier la syntaxe avant d'envoyer : copier `app.js` en `a.mjs` puis `node --check a.mjs`.

## Carte de `app.js` (repères `// ---- …`)
| Ligne ≈ | Section |
|---|---|
| 19 | réglages (`CFG`, `DEF`, valeurs par défaut) |
| 41 | systèmes (`SYS`, `SYSID`) et chargement paresseux |
| 72 | scène : renderer, lumières, boucle `loop()`, `invalidate()`, résolution dynamique `lowRes()` |
| 138 | données : `ingest()`, `parts`, noms, options (fascias, bourses…) |
| 328 | pointage (`pick`) |
| 376 | sélection, visibilité, annulation (`select`, `snap`/`restore`, `CURV` = filtre de la vue active) |
| 401 | reliefs osseux (pastilles numérotées, halos colorés) |
| 482 | insertions musculaires · 502 carte · 522 fiches muscles |
| 689 | coupe (plans de section + bouchons stencil colorés par structure) |
| 757 | vue éclatée locale et progressive |
| 786 | vues par région / articulation (`VIEWS`, `applyView`) |
| 835 | panneau de réglages (`ROWS`) |
| 870 | ancien quiz (modes fiche/clic) |
| 974 | navigateur Atlas / Quiz (pages par région, groupes anatomiques `GRP`, vignettes) et quiz « Désigner » (`pqStart`) |
| 1275 | quiz « Écrire le nom » : scène dédiée (`tyStart`, `tyFrame`), sons, clavier à l'écran |

## Fonctions déjà en place (ne pas refaire)
- Noms français / latins / anglais (réglages), fiches muscles propres, coupe avec surfaces colorées, éclatement local progressif.
- Onglets **Atlas** et **Quiz** dans le menu de gauche, modelés sur Anatomy Learning. **Atlas** = zones (épaule, coude, poignet, hanche, genou, cheville, régions, appareils, « Mes vues »), puis accès « par système ».
- Quand une vue est active (ex. Épaule) et qu'on ajoute « Muscles », seuls les muscles de la zone s'affichent (bandeau « Vue : … » pour retirer le filtre).
- Quiz « Désigner » : pastilles ou survol de la structure (réglage). Quiz « Écrire le nom » : mode séparé (choix « Mode » en haut de la page Quiz), fond clair, modèle fantôme, cible en bleu (artères en rouge), carrousel avant/après, sons de clavier, 3 niveaux d'aide.
- Un seul membre affiché (le droit), couleurs par famille d'os, lazy-loading des fascias, bourses, cerveau profond, segments du foie.
- Reliefs osseux : cliquer dans le vide ferme le menu mais garde les pastilles affichées (Échap les ferme).
- Hors-ligne (service worker).

## Pièges connus
- `renderer.setSize()` efface le canevas : toujours appeler `invalidate()` ensuite (c'était la cause d'un cadre gris au clic).
- Dans `ingest()`, les maillages glTF sont aplatis (`root.attach(m)`) : sinon masquer un parent masque ses enfants.
- Le quiz « Écrire le nom » utilise sa propre scène : une seule surface de modèle fantôme (pré-passe de profondeur) et la cible sur une couche à part ; ne pas réintroduire de transparence empilée (effet « fouillis »).
- Les caches (navigateur, service worker, `localStorage` pour les vignettes `atlas.th7` et les scores `atlas.pq`) donnent souvent l'impression qu'un correctif ne marche pas : vider d'abord.
- Les noms de structures finissent par `.r` / `.l` (côté droit / gauche) ; seul le côté droit est affiché.
- Pas de réseau pour les outils de build : ne rien ajouter qui exige `npm install`.

## À faire / idées
- Tester sur un vrai téléphone (jamais fait) : tactile, clavier à l'écran, performance.
- Zones osseuses délimitées à refaire à la main d'après Visible Body (l'utilisateur fournira l'appli ; aucune extraction de données ou de textures).
- Descriptions précises de chaque structure, meilleures textures peau et cheveux, vignettes contextuelles (muscles voisins) pour les quiz.
- Système lymphatique : fait à la main, imprécis — ne pas y toucher sauf demande.

## Règles de collaboration
- Une modification précise à la fois ; renvoyer la fonction complète modifiée.
- Faire un commit avant chaque changement ; tester dans le navigateur (console F12) à chaque étape.
- Ne pas copier de contenu protégé (textures, données, interface d'un autre logiciel) : s'en inspirer seulement.
