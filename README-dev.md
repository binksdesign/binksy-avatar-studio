# Avatar Norbert × Binks — notes de développement

Recréation + extension du générateur « OpenBot plush avatars » de Norbert Bodziony, pour Graphiste Binks.
Aucun code ni asset d'OpenBot copié : tout est réécrit et procédural (formes, yeux, fourrure, accessoires).

## Livrables
| Fichier | Rôle |
|---|---|
| `Avatar-Norbert-Binks.html` | l'app complète, un seul fichier (817 Ko, 240 Ko gzip), hors ligne, marche en `file://` (three.js, fflate, gifenc, muxers, Lucide intégrés) |
| `avatar-bot.js` | composant `<avatar-bot>` autonome (35 Ko minifié, 14 Ko gzip) |
| `component-demo/` | `demo.html` + `avatar-bot.js` + `pack.js` (pack d'exemple exporté par l'app : 3 agents peluche × 3 états) |
| `NOTES.md` | ce document |

Build : `python3 build.py` (icônes Lucide → `src/icons.gen.js`, composant → `avatar-bot.js` + `src/component.gen.js`, app → esbuild IIFE minifié inséré dans `src/index.html`).
Sources dans `src/`, composant dans `component/avatar-bot.src.js`, tests dans `test/`, captures dans `shots/`.

## Architecture (`src/`)
| Fichier | Rôle |
|---|---|
| `schema.js` | **source unique** : personnages, 11 formes, 17 expressions, 12 couleurs, 8 états, 5 transitions, 10 rendus + réglages, 25 accessoires, bouches, rôles/noms ; `validateConfig()` (strict, corrige et signale), `jsonSchema()`, `listOptions()`, aléatoire reproductible, `makeTeam()` |
| `shapes.js` | champs de distance 2D des silhouettes (polygones arrondis, superellipse, nuage en union lisse…) → corps 3D « coussin » (dalle arrondie + bombé radial sans arêtes), lancer de rayons, ancres, contour SVG lissé |
| `pose.js` | fonction **pure** pose(cfg, u) → yeux, corps (respiration, squash), extras (points, badge, z), bouche ; u ∈ [0,1[ → boucles parfaites ; utilisée par le 3D, le Flat et le composant |
| `face.js` | yeux pilule / arc + paupières + bouche → primitives communes (shader 3D et SVG) |
| `facegl.js` | patch du visage (triangles avant du maillage du corps, décalés) + shader : yeux blancs bombés et brillants (normale reconstruite depuis le champ de distance), ombre portée, bouche, joues ; mêmes uniformes pour masquer la fourrure autour des yeux |
| `fur.js` | **peluche = shell texturing** (repris et adapté d'Avatar Studio v5, code Binks) : N coques instanciées, mèches par bruit cellulaire 3D, touffes, duvet, occlusion racine, pointes claires, lustre (sheen), alpha-to-coverage MSAA, composition « par-dessus » pour l'alpha d'export ; la fourrure s'écarte autour des yeux/bouche |
| `materials.js` | styles : Lisse brillant, Mat, Verre/gelée (coque translucide + cœur lumineux), Argile (empreintes par bruit dans la normale), Chrome, Velours (sheen), Toon (dégradé à paliers + contour coque inversée), Vinyle ; dissolution injectée dans toutes les matières |
| `accessories.js` | 25 accessoires 3D procéduraux posés sur les ancres de la forme (sommet, côtés, surface par lancer de rayons, visage), cumulables, colorables, x/y/z/échelle/rotation |
| `flat.js` | rendu **Flat 2D en vrai SVG** (corps, ombrage, contour, yeux, bouche, extras, accessoires 2D) |
| `engine.js` | three.js r170 : icosphère projetée sur chaque forme (**même topologie** → morphing « warp » direct), transitions flip/pop/warp/dissolve/cut, vie (regard, clignements, respiration), 2 renderers (aperçu + hors écran pour vignettes, lignes « Dans l'app » et exports), cadrage auto (accessoires, points de réflexion) |
| `exporters.js` | PNG, JPEG, SVG, GIF (gifenc), WebP animé, APNG, MP4/WebM (WebCodecs), WebM **alpha** (2ᵉ encodeur VP9 sur l'alpha, webm-muxer patché), planches de sprites + JSON Phaser + CSS, pack d'icônes + favicon.ico, lots ZIP + manifest, pack composant |
| `ai.js` | OpenRouter (modèles, chat JSON, erreurs claires, nouvel essai si JSON invalide, validation par le schéma) ; logo/photo → avatar (vision) ; repli local : palette k-moyennes + forme devinée |
| `webmcp.js` | WebMCP (`document.modelContext.registerTool`, repli `navigator.modelContext` / `provideContext`), détection de fonction |
| `audio.js` | état « Parle » : niveau RMS du micro ou d'un fichier audio |
| `store.js` | sauvegarde IndexedDB (repli localStorage) |
| `app.js` | interface (onglets Avatar · Rendu · Accessoires · Équipe · IA · Export), historique, sauvegarde auto, lien, JSON, presets, `window.avatarAPI` |

## Fonctionnalités (comptes)
- 6 personnages (Nexo, Vita, Cloe, Malu, Yoshi, Fred), 11 formes, 17 expressions (« Visage du perso » compris), 12 pastilles + couleur libre + palette de marque, 8 états animés (Repos, Clin d'œil, Yeux grands, Réfléchit, Notification, Dort, Échec, Parle), 5 transitions, taille des yeux, couleur des yeux, bouche optionnelle (8 styles + auto).
- 10 rendus (Peluche, Lisse brillant, Mat, Verre/gelée, Argile, Chrome, Velours, Toon, Vinyle, Flat SVG) avec leurs réglages (24 curseurs au total).
- 25 accessoires. Mode équipe 2–8 agents. Aléatoire (reproductible par graine). Mes avatars (presets). Annuler/rétablir (150 niveaux, Ctrl/Cmd+Z, Ctrl+Maj+Z).
- Vignettes du panneau rendues par le vrai moteur (style courant).
- « Dans l'app — survole une ligne » : lignes à 24 / 32 / 40 px, animées au survol, case « toujours animer » ; avec une équipe, ce sont ses agents.
- 12 types d'export : PNG, JPEG, SVG, GIF, WebP, APNG, MP4, WebM, Sprites, Icônes, Lot ZIP, Pack composant.

## Configuration JSON (schéma)
`window.avatarAPI.schema()` ou Réglages → « Copier le schéma JSON ». Exemple :
```json
{ "v": 1, "name": "Juri", "character": null, "shape": "pentagon", "color": "#1e2a78", "expression": "proud", "state": "idle",
  "transition": "pop", "eyeSize": 1.1, "eyeColor": "#f6f2ea", "render": { "style": "plush", "p": { "length": 0.08, "density": 1.15 } },
  "accessories": [{ "type": "glasses", "color": "#f5c542", "x": 0, "y": 0, "z": 0, "scale": 1, "rot": 0 }], "mouth": { "on": true, "style": "smile", "color": "#2a161b" } }
```
Validation : valeurs hors bornes ramenées, synonymes FR/EN acceptés (« rond », « fluffy », « joyeux »…), champs inconnus ignorés, chaque correction renvoyée dans `warnings`.

## API navigateur
`window.avatarAPI` : `getAvatar()`, `setAvatar(config, merge=true)`, `setState(s)`, `setExpression(e)`, `addAccessory(a)`, `randomize(seed?)`, `exportAvatar({format, size, state})` (data URL si < 1,5 Mo, sinon téléchargement), `generateWithAI({prompt, mode: single|team|variants, n})`, `listOptions()`, `schema()`, `encodeHash()`, `decodeHash()`, `undo()`, `redo()`, `webmcp` (statut).
WebMCP : 9 outils `get_avatar, set_avatar, list_options, set_state, set_expression, add_accessory, randomize, export_avatar, generate_with_ai`, enregistrés avec `document.modelContext.registerTool(tool, {signal})` (forme actuelle du brouillon webmachinelearning/webmcp), repli `navigator.modelContext.registerTool` puis `provideContext({tools})`. Résultats au format `{ content: [{ type: 'text', text }] }`, erreurs `isError: true`.

## IA (OpenRouter)
Clé saisie dans Réglages, stockée en IndexedDB sur l'appareil, jamais dans le fichier. Liste des modèles via `GET /api/v1/models` (filtre texte + « vision », prix affiché). Défaut : premier disponible parmi `google/gemini-2.5-flash-lite`, `google/gemini-2.5-flash`, `openai/gpt-4.1-mini`, `openai/gpt-4o-mini`.
`response_format: json_object` (retiré automatiquement si le modèle le refuse) ; JSON extrait même entre balises ```json ; un second essai si la réponse est invalide ; erreurs 401/402/403/404/408/429/5xx/réseau traduites en français. Palette de marque verrouillée transmise au modèle et imposée après coup (couleur la plus proche).

## Lien partageable
`#a=` + JSON compressé (zlib, fflate) en base64url (~370 caractères). Chargé au démarrage et sur `hashchange` (annulable).

## Sauvegarde automatique (règle permanente)
Tout est enregistré en IndexedDB (`avatar-norbert-binks` / `kv` / `state`) 220 ms après chaque changement, et à `pagehide` / onglet masqué : avatar, équipe (agents, noms, rôles, agent actif), presets, réglages (clé API, modèle, regard, toujours animer, palette de marque, onglet, pause), réglages d'export (format, taille, i/s, transparence, fond, lot), prompt IA et variantes, accessoire sélectionné. Testé : rechargement → état identique (comparaison JSON stricte).

## Composant `<avatar-bot>`
```html
<script src="avatar-bot.js"></script><script src="pack.js"></script>
<avatar-bot pack="equipe" agent="nexo" state="idle" size="32" animate="hover"></avatar-bot>
<script>document.querySelector('avatar-bot').setState('thinking')</script>
```
- Joue les planches de sprites (WebP, repli PNG) exportées par « Pack composant » ; `pack.js` embarque les images (marche en `file://`) ; sur serveur http : `<avatar-bot src="manifest.json">`.
- Attributs : `state`, `size` (24 px OK, rendu net en Retina avec des sprites 80 px), `animate="always|hover|none"`, `agent`, `pack`, `expression`. Pause hors écran (IntersectionObserver), respecte `prefers-reduced-motion`.
- Mode direct optionnel `live` (+ `follow`) : même dessin en SVG vectoriel, calculé en temps réel (expressions et états dynamiques via `setExpression`, `setState`).

## Format des lots (`manifest.json`)
`format: avatar-norbert-pack`, `states` (clés anglaises : idle, wink, wide_eyes, thinking, notification, asleep, failed, speaking + durée de boucle), `agents`, et par fichier : chemin `agent/état/expression.ext`, état, expression, format, mime, taille, animé, boucle, durée, i/s, images, transparent, planche (cols/rows), octets.

## Open source (licences)
| Projet | Licence | Usage |
|---|---|---|
| three.js r170 (+ RoomEnvironment, BufferGeometryUtils) | MIT | embarqué |
| fflate 0.8.3 | MIT | ZIP, zlib du lien |
| gifenc 1.0.3 | MIT | GIF (quantification PnnQuant, transparence 1 bit) |
| mp4-muxer 5.2.2, webm-muxer (patch alpha d'Avatar Studio) | MIT | vidéo |
| Lucide | ISC | icônes d'interface |
| APNG / WebP animé / ICO | code maison (Avatar Studio, Binks) | assemblage de conteneurs |
| WebMCP (webmachinelearning/webmcp) | W3C CG | forme de l'API suivie, rien d'embarqué |
UPNG.js n'a pas été nécessaire (APNG assemblé à partir des PNG du navigateur).

## Tests (`test/`, Playwright + Chromium headless SwiftShader, commandes ≤ 2 min, séries longues en arrière-plan)
- `d0.py` planches moteur (formes, styles, expressions, états, accessoires) → `shots/sheet-*.png`, `shots/d_*.png`.
- `t1.py` / `t2.py` captures desktop (tous les onglets) et mobile iPhone (390×844, DPR 2, tactile).
- `t3.py` fonctionnel : WebMCP simulé (9 outils enregistrés, appels réels), OpenRouter simulé (route Playwright : modèles, génération simple/équipe/variantes, erreur 401, sans clé), lien `#a=`, annuler/rétablir, **sauvegarde auto + rechargement** (égalité stricte), chargement par lien.
- `t4.py` exports réels validés par `ffprobe` / PIL / zip (voir résultats ci-dessous).
- `t5.py` composant : démo à 24/32/40 px, `setState`, mode direct, `file://`.
- `t6.py` app en `file://` (aucune requête réseau, aucune erreur, Flat + export SVG, rechargement) · `t7.py` transitions (planche) · `t8.py` équipe + lignes « Dans l'app » · `t9.py` rechargement immédiat (copie de secours).

## Résultats (2026-11-02/03, build final)
- 11 formes × 10 rendus = 110 combinaisons rendues sans erreur → `shots/sheet-shapes-x-renders.png`. 17 expressions, 8 états, 25 accessoires, 5 transitions rendus sans erreur (`shots/d_expr.png`, `shots/sheet-accessories.png`, `shots/sheet-transitions.png`).
- WebMCP simulé : 9 outils enregistrés via `registerTool`, appels réels OK (set_state, set_expression, add_accessory + erreur propre pour un type inconnu, list_options, set_avatar avec corrections signalées, randomize reproductible par graine, export_avatar → data URL PNG, get_avatar).
- OpenRouter simulé : sans clé → message clair ; simple (JSON entre balises ``` + accessoire inconnu ignoré et signalé), équipe de 3 (noms + rôles), 4 variantes, 401 → « Clé API refusée », en-tête `Authorization: Bearer`, `response_format` envoyé.
- Sauvegarde auto : rechargement → avatar, équipe, presets, réglages (clé, modèle, palette, onglet, toujours animer), export, prompt **identiques** ; rechargement immédiat sans délai → restauré (copie de secours) ; lien `#a=` → avatar chargé ; annuler/rétablir OK.
- Exports validés : PNG 256 RGBA alpha ; JPEG ; SVG (Flat) ; GIF 96 px 27 images transparent ; APNG 27 images RGBA ; WebP animé 27 images RGBA ; MP4 27 images (VP9 dans ce Chromium) ; WebM VP9 `alpha_mode=1`, image décodée avec alpha 0–255 ; planche de sprites 45 images + JSON + CSS ; icônes 16…512 + favicon.ico (16/32/48) ; lot équipe (2 agents × 2 états × PNG/WebM/GIF/sprite = 16 fichiers + manifest cohérent) ; pack composant (3 agents peluche × 3 états, WebP) → `component-demo/`.
- Composant : 24 instances, tailles 24/32/40 px réelles (canvas ×2), images dessinées, `setState` OK, survol anime, mode direct en SVG, `setExpression` OK, aucune erreur console, en `file://`.
- Captures : `shots/ui-desktop.png`, `shots/ui-mobile.png` (iPhone 390×844), `shots/ui-tab-*.png`, `shots/ui-flat-file.png`, `shots/component-demo.png`, `shots/sheet-render-styles.png`, `shots/sheet-accessories.png`.

## Limites connues
- Le Chromium de test n'a pas d'encodeur H.264 : le MP4 testé contient du VP9 (Chrome/Edge/Safari réels encodent en H.264). Le WebM alpha n'est lisible avec transparence que dans Chrome/Firefox (Safari ignore l'alpha VP9).
- Safari ne sait pas encoder le WebP via canvas : WebP animé et sprites WebP basculent en message d'erreur / PNG ; utiliser APNG ou GIF sur iPhone.
- Fourrure : temps réel par coques — très bon de face, silhouette légèrement « en couches » à fort zoom ; l'aperçu mobile est allégé (22 coques, définition réduite), les exports sont en pleine qualité (56 coques, sur-échantillonnage ×2).
- Tout a été mesuré en rendu logiciel (SwiftShader) : performance réelle sur GPU non mesurée ici. Exports de peluche lents en logiciel (≈ 0,3–1 s/image).
- WebMCP n'est exposé par aucun navigateur stable à ce jour : testé avec un `document.modelContext` simulé.
- OpenRouter testé avec des réponses simulées (pas de clé réelle dans le bac à sable).
- Accessoires génériques : sur certaines formes (triangle, goutte) les chapeaux suivent l'ancre du sommet mais peuvent demander un réglage manuel (curseurs position/taille).
