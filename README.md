# Binksy Avatar

Avatars d'agents procéduraux (peluche, 3D, Flat SVG) — outil web autonome de Binks Design, servi sur avatar.binksdesign.com.
Inspiré d'OpenBot (Norbert Bodziony) ; tout le code et les formes sont réécrits et procéduraux.

- `index.html` : l'app complète, un seul fichier, hors ligne, marche en `file://` (three.js, fflate, gifenc, muxers, Lucide intégrés).
- `composant/` : composant `<avatar-bot>` autonome + démo + pack de sprites d'exemple (3 agents peluche × 3 états).
- `README-dev.md` : notes de développement (architecture, API, tests, limites connues).

## En bref
- 6 personnages, 11 formes, 17 expressions, 8 états animés, 5 transitions, 10 rendus (Peluche, Lisse brillant, Mat, Verre, Argile, Chrome, Velours, Toon, Vinyle, Flat SVG).
- **87 accessoires** style « 3D icon » (volumes épais, biseaux ronds, 2–3 couleurs), en 6 catégories : Chapeaux (casquette, snapback, trucker, casquette à l'envers, bob, bob bicolore, bonnets, cagoule, capuche, casques…), Tête & oreilles, Visage, Bouche, Cou & corps (gilet pare-balles avec écusson, écharpe tricot, cape, sac à dos…), Fun / props. Couleur principale + accent, écusson texte court, position / taille / rotation ; ajustés aux 11 formes.
- Équipes de 2 à 8 agents, IA (OpenRouter, clé gardée sur l'appareil), WebMCP, lien partageable, 12 formats d'export (PNG, SVG, GIF, WebP, APNG, MP4, WebM alpha, sprites, icônes, lots ZIP, pack composant).
- Sauvegarde automatique dans le navigateur (IndexedDB `binksy-avatar`) ; les données de l'ancienne version (« Avatar Norbert × Binks ») sont reprises automatiquement au premier lancement.

## Composant

```html
<script src="composant/avatar-bot.js"></script>
<script src="composant/pack.js"></script>
<avatar-bot pack="equipe" agent="nexo" state="idle" size="32" animate="hover"></avatar-bot>
```

Site : avatar.binksdesign.com
