# binksy-avatar-studio

Avatars peluche procéduraux pour agents — outil web autonome de Binks Design, servi sur Cloudflare Pages.

- `index.html` : l'app complète, un seul fichier, hors ligne, marche en `file://` (three.js, fflate, gifenc, muxers, Lucide intégrés).
- `composant/` : composant `<avatar-bot>` autonome + démo + pack de sprites d'exemple.
- `README-dev.md` : notes de développement (architecture, API, tests, limites connues).

## Composant

```html
<script src="composant/avatar-bot.js"></script>
<script src="composant/pack.js"></script>
<avatar-bot pack="equipe" agent="nexo" state="idle" size="32" animate="hover"></avatar-bot>
```

Site : avatar.binksdesign.com
