# YunoParty

**Regardez, parlez et jouez entre amis.** Un lien, et vos amis sont dans la party : voix, chat, film synchronisé,
navigateur partagé que tout le monde contrôle en même temps, et quelques jeux. Site statique, en **français et en
anglais**, Supabase pour les comptes, les parties et le temps réel. **Tout le site tient aussi dans un seul fichier
`index.html`** (voir ci-dessous) : double-clic pour l'ouvrir, ou envoyez ce seul fichier chez n'importe quel hébergeur.

> English, in two lines: YunoParty is a static website (FR + EN) where friends join a "party" through a shareable
> link to talk (voice), chat, watch a synced movie, control **one shared browser together**, and play small games.
> The whole site is also built as **one self-contained `index.html`** (double-click to open it, or upload that single file
> to any static host; add `?mode=demo` to try it without a backend). For the Supabase SQL, use the in-site "Mise en place" page (`#/setup`).

---

## Ce que fait le site

| | |
|---|---|
| **Comptes** | Inscription / connexion par e-mail + mot de passe, lien magique, ou **invité** (juste un pseudo). Avatar (24 illustrations SVG) et couleur au choix. |
| **Parties** | Créer une party (icône SVG au choix), partager le **lien** ou le **code** à 6 caractères. Tableau de bord « Mes parties », réglages, exclusion de membres, transfert de l'hôte, quitter ou supprimer. |
| **Voix** | Vocal de groupe en WebRTC (micro, sourdine, volume par personne, anneau lumineux autour de celui qui parle) et partage d'écran (avec le son de l'onglet quand le navigateur le permet). |
| **Chat** | Messages en direct avec historique, liens cliquables, **stickers et réactions SVG** (`:rx-…:`), réactions flottantes visibles de tous, indicateur « écrit… ». |
| **Regarder ensemble** | Lecteur synchronisé : YouTube, Vimeo, Twitch, fichiers vidéo / flux HLS (`.m3u8`), **fichier local** (chacun ouvre le même), page web intégrée, partage d'écran. **Tout le monde peut tout contrôler** (lecture, pause, avance, vitesse, changement de source) ; chaque client corrige sa dérive. |
| **Navigateur partagé** | Option « Navigateur partagé » : un vrai Chromium (sur *votre* petit serveur, dossier `server/`) dont l'écran est diffusé à tous. **Tout le monde clique, tape et fait défiler en même temps**, chacun voit en direct les curseurs colorés des autres. Format ordinateur ou **téléphone**, gestes tactiles sur mobile. Facultatif, voir [server/README.md](server/README.md). |
| **Jeux** (7) | Morpion, Puissance 4 · *Tu préfères ?*, *Qui est le plus…*, *Quiz express* · *Duel de réflexes* · *La roue*. Tout est synchronisé, avec spectateurs, reconnexion et rechargement de page. |
| **Personnalisation** | 6 thèmes (4 sombres : Néon, Coucher de soleil, Océan, Forêt ; 2 clairs : Bonbon, Citron) + teinte libre, 4 fonds animés (Aurore, Étoiles, Grille, aucun), animations (activées / auto / réduites), densité, sons, langue FR/EN. Les préférences suivent le compte. |
| **Mobile** | Interface pensée pour téléphone : navigation du bas (Regarder · Chat · Jeux · Personnes), zones tactiles confortables, aucun défilement horizontal, navigateur partagé avec gestes tactiles. |
| **Zéro emoji** | Avatars, icônes de party, stickers, réactions, drapeaux, icônes de jeux : **tout est dessiné en SVG**. Un test (`tools/check-no-emoji.mjs`) empêche d'en réintroduire. |

## Le fichier unique `index.html`

Le dossier du projet est le **code source** (modules ES répartis en plusieurs fichiers : il lui faut un petit serveur HTTP).
Pour ne manipuler qu'**un seul fichier**, un outil fusionne HTML + CSS + tout le JavaScript + les illustrations SVG dans
**`dist/index.html`** (≈ 650 Ko ; livré aussi à part, sous le nom `index.html`).

- **Double-clic** : il s'ouvre directement depuis le disque (`file://`), sans serveur ni dossier à côté. Sans Internet, ou avec
  `?mode=demo` ajouté à l'adresse (`file:///…/index.html?mode=demo`), il tourne en mode démo : tout reste dans le navigateur.
- **Mise en ligne** : envoyez ce seul fichier, nommé `index.html`, chez Netlify (glisser-déposer), Cloudflare Pages, Vercel,
  GitHub Pages… Vos amis n'ont plus qu'un lien à ouvrir. **HTTPS obligatoire** pour le micro, le partage d'écran et YouTube.
- **Inviter des amis demande une adresse en ligne** : depuis un fichier local, le lien d'invitation ne marche que sur votre
  ordinateur (la fenêtre « Inviter » le signale). Les e-mails de confirmation et les liens magiques renvoient vers le
  « Site URL » réglé dans Supabase, pas vers le fichier.
- **Le refabriquer** après une modification : `node tools/build-single.mjs` (demande TypeScript : `npm i -g typescript`) →
  `dist/index.html`. Les réglages (`js/config.js` : projet Supabase, serveurs ICE, navigateur cloud) sont ceux du source :
  modifiez-les, puis refabriquez. `node tests/e2e/single-file.test.mjs` vérifie le résultat ouvert en `file://`.

## Essayer tout de suite (mode démo, sans rien installer)

Le plus simple : **ouvrez le fichier unique `index.html` dans votre navigateur** (et ajoutez `?mode=demo` à l'adresse pour
rester en démo même avec Internet).

Depuis le dossier source, les modules JavaScript demandent un vrai serveur HTTP (pas `file://`). N'importe lequel convient :

```bash
cd yunoparty
python3 -m http.server 8080        # ou : npx serve .
```

Ouvrez **http://localhost:8080/?mode=demo**. En mode démo tout reste dans votre navigateur (base locale + `BroadcastChannel`) :
**ouvrez la page dans plusieurs onglets pour jouer plusieurs amis** (chat, jeux, lecteur synchronisé, présence). La voix
et le partage d'écran fonctionnent aussi entre onglets.

Sans `?mode=demo`, le site essaie Supabase et retombe sur le mode démo s'il ne répond pas (`mode: 'auto'` dans `js/config.js`).

## Mise en ligne avec Supabase

Le projet Supabase `cuqwoyqheirvfuuqxylg` est déjà renseigné dans `js/config.js` (URL + clé *publishable*, faite pour être
publique : la sécurité repose sur les règles RLS). **Ne mettez jamais la clé `secret` / `service_role` dans le site.**

### 1. Appliquer les migrations SQL (à faire une fois)

La migration 1 (`core_schema`) est déjà en place. **Les migrations 2, 3 et 4 n'ont pas pu être appliquées automatiquement** :
Supabase demande une confirmation pour ce type de SQL, et dans la session de développement cette confirmation était annulée
d'office en moins d'une seconde (essayé plusieurs fois, avec l'outil de migration puis avec l'exécution SQL, y compris après
un accord explicite dans le chat). Je n'ai volontairement rien contourné. À faire une seule fois, à la main (≈ 30 secondes).
**Rien à ouvrir ni à télécharger : tout se fait depuis le site**, sur la page **« Mise en place »** :

1. Ouvrez le site → en bas de la page d'accueil, lien **« Mise en place »** (ou ajoutez `#/setup` à l'adresse).
2. **Copier le SQL** (si le navigateur bloque la copie, le texte s'affiche déjà sélectionné : Ctrl/Cmd + C).
3. **Ouvrir l'éditeur SQL** : le bouton mène droit à l'éditeur de votre projet
   (<https://supabase.com/dashboard/project/cuqwoyqheirvfuuqxylg/sql/new>, ou Dashboard → **SQL Editor** → *New query*).
   Collez → **Run**. Si l'éditeur avertit d'opérations « destructives » : le script remplace des fonctions et contient des
   `drop policy if exists` sur ses propres règles, aucune donnée n'est supprimée → confirmez.
4. Revenez sur la page « Mise en place » : elle **se vérifie toute seule** (au retour sur l'onglet) et affiche *En place* ou
   *À appliquer* pour chaque migration (il faut être connecté ; un compte invité suffit).
5. Rejouable sans risque : l'exécuter deux fois ne casse rien. Résultat attendu dans l'éditeur : *Success. No rows returned*.

Le même SQL est dans le projet : `supabase/paste-in-sql-editor.sql` (migrations 2 + 3 + 4 réunies, dans l'ordre). La page du
site en est générée (`node tools/build-setup-sql.mjs`, vérifié par `--check`).

**Si vous n'en appliquez qu'une, prenez la n° 2** : c'est elle qui fait respecter l'exclusion et le verrouillage d'une party.

| Migration | Apporte | Sans elle |
|---|---|---|
| `…011000000_secrets_moderation` | **exclusion définitive** d'un membre et **verrouillage** de la party appliqués par la base, état partagé « dernier écrivain gagne », secrets privés (informations cachées dans les jeux) | **un membre exclu peut revenir avec le lien et « Verrouiller » n'empêche personne d'entrer** (l'exclusion retire seulement la personne ; l'état partagé retombe sur une écriture simple) ; les secrets privés des jeux à rôles cachés sont indisponibles (les 7 jeux livrés n'en ont pas besoin) |
| `…011120000_party_tools` | « Mes parties » en une requête (effectif, dernière activité), réglages de party, transfert d'hôte, quitter | le site retombe sur des requêtes simples autorisées par RLS (moins d'informations affichées) |
| `…017000000_svg_defaults` | avatar et icône de party **par défaut** en SVG côté base | le site traduit les anciennes valeurs (emoji) en illustrations à l'affichage ; rien de visible ne change |

### 2. Réglages Authentication (Dashboard → Authentication)

- **Providers → Email** : activé. Pour des amis, vous pouvez désactiver *Confirm email* (sinon chaque inscription attend un
  e-mail ; le SMTP par défaut de Supabase est très limité en nombre d'envois — configurez le vôtre pour une vraie mise en ligne).
- **Sign In / Providers → Allow anonymous sign-ins** : **activé**, c'est ce qui permet le mode *Continuer comme invité*.
- **URL Configuration** : *Site URL* = l'adresse publique du site ; ajoutez-la (et `http://localhost:8080`) aux *Redirect URLs*
  (liens magiques, confirmation d'e-mail, mot de passe oublié).
- **Realtime** : le site utilise des canaux **privés** (autorisés par les règles RLS de la migration 1). Rien d'autre à régler.

### 3. Héberger le site

N'importe quel hébergeur statique : Netlify, Vercel, Cloudflare Pages, GitHub Pages… **Le plus simple : envoyez le seul fichier
`index.html`** (voir « Le fichier unique »). Sinon envoyez le dossier (sans `node_modules`, `tests/`, `server/`).
Le routage utilise le `#` : **aucune règle de réécriture à prévoir**.
**HTTPS est obligatoire** pour le micro, le partage d'écran et les liens d'invitation (c'est automatique sur ces hébergeurs).

### 4. (Facultatif mais conseillé) Ne dépendre d'aucun CDN

`supabase-js` est chargé dans le navigateur depuis la première source qui répond (`sdk` dans `js/config.js`). Pour ne
dépendre que de votre site, téléchargez le fichier et déposez-le :

```bash
mkdir -p vendor
curl -L -o vendor/supabase.js https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.58.0/dist/umd/supabase.js
```

(Le dossier `vendor/` n'existe pas dans l'archive : mon environnement de développement n'avait pas accès au réseau pour le
télécharger. Le lecteur HLS et l'API YouTube viennent de leur CDN officiel ; les polices, de Google Fonts. Le fichier unique
contient tout le code du site, mais ces quatre éléments externes (`supabase-js`, hls.js, YouTube, polices) restent chargés
depuis leur CDN.)

### 5. (Facultatif) Voix derrière un réseau strict

La voix et le partage d'écran utilisent WebRTC avec des serveurs **STUN** publics (`iceServers` dans `js/config.js`) : cela
suffit dans la plupart des cas (box, Wi-Fi). Derrière certains réseaux (4G, entreprises, universités) il faut un relais
**TURN** : ajoutez-le dans `iceServers` (`{ urls: 'turn:…', username: '…', credential: '…' }`). Ce n'est pas fourni.

### 6. (Facultatif) Navigateur partagé « cloud »

Voir **[server/README.md](server/README.md)** : un petit serveur Node + Chromium (Docker fourni), authentification par jeton
Supabase + appartenance à la party, protection SSRF, un contexte isolé par party. Puis renseignez
`cloud: { url: 'wss://cloud.votre-site.example' }` dans `js/config.js`. Sans serveur, le bouton affiche simplement comment l'installer.

## Utilisation

1. Créez un compte (ou continuez en invité), puis **Créer une party**.
2. **Partagez le lien** (bouton partager) : en l'ouvrant, vos amis arrivent sur une page d'invitation, créent un compte ou entrent comme invités, et sont dans la party.
3. Dans la party : *Regarder* (lien, fichier, partage d'écran ou navigateur partagé), *Chat*, *Jeux*, *Personnes*. Le bouton « Rejoindre le vocal » est en bas.

## Structure du projet

```
index.html              point d'entrée du code source (modules ES, routage par #)
dist/index.html         LE FICHIER UNIQUE (généré par tools/build-single.mjs : tout le site dedans)
css/                    système de design (thèmes par teinte --h), pages, salle, jeux (un fichier par catégorie)
js/
  main.js app.js        démarrage, routeur, préférences
  pages/                accueil, connexion, invitation, party, page « Mise en place » (setup.js)
  room/                 chat, lecteur synchronisé, voix (WebRTC), personnes, dock, navigateur cloud, noyau temps réel
  backend/              Supabase (backend.js) et base locale de démo (localdb.js) : UN SEUL chemin de code pour les deux
  games/                moteur d'activités (kit.js, api.js, lib.js), jeux par catégorie, registre généré
  glyphs.js icons.js    illustrations SVG : avatars, icônes de party, stickers, drapeaux, icônes
  data/setup-sql.js     le SQL de la page « Mise en place » (généré depuis supabase/paste-in-sql-editor.sql)
server/                 navigateur partagé « cloud » (facultatif) + son README et son Dockerfile
supabase/migrations/    schéma SQL, 4 migrations (+ paste-in-sql-editor.sql = les migrations 2 à 4 en un fichier)
docs/CONTRACT.md        comment écrire une nouvelle activité (jeu, outil)
tests/ tools/           tests automatiques et vérifications (voir plus bas) ; tools/build-single.mjs fabrique le fichier unique
                        (avec tools/system-runtime.js, son petit chargeur de modules), tools/build-setup-sql.mjs le SQL de la page
```

### Ajouter un jeu

Lisez `docs/CONTRACT.md`, copiez un jeu existant (`js/games/boards/tictactoe.js` pour un jeu au tour par tour,
`js/games/quiz/` pour un jeu à vote), puis `node tools/build-registry.mjs` : le catalogue du site est régénéré et l'activité
est vérifiée (pas d'`innerHTML`, pas de réseau, pas d'emoji, icônes SVG existantes).

## Vérifier que tout fonctionne

Les tests utilisent Playwright (déjà installé dans l'environnement de développement ; chez vous :
`npm i -D playwright && npx playwright install chromium`).

```bash
node tools/check-no-emoji.mjs                  # aucun emoji dans l'interface
node tools/check-i18n.mjs                      # toutes les clés FR/EN existent
node tools/build-registry.mjs --check          # catalogue des activités à jour et valide
node tools/build-setup-sql.mjs --check         # le SQL de la page « Mise en place » est celui du fichier .sql
node --test tests/unit/core.test.mjs tests/unit/system-runtime.test.mjs tests/games/poll.unit.test.mjs \
            tests/games/reflex.unit.test.mjs tests/games/wheel.unit.test.mjs tests/games/connect4.unit.test.mjs \
            tests/server/guard.unit.test.mjs
node tests/games/wyr.test.mjs                  # un scénario par jeu (plusieurs joueurs simulés) :
                                               #   wyr, mostlikely, trivia, reflex, wheel, tictactoe, connect4
node tests/smoke.mjs                           # toutes les activités : FR/EN, écran de téléphone, coupure réseau, arrivée tardive
node tests/e2e/app.test.mjs                    # le site entier en mode démo, plusieurs amis (comptes, party, chat, voix, lecteur, jeux)
node tests/e2e/glyphs.test.mjs                 # zéro emoji, anciennes valeurs de base, mobile 390 px
node tests/server/cloud.test.mjs               # serveur du navigateur cloud de bout en bout
node tests/e2e/cloud.test.mjs                  # site + serveur : ordinateur et téléphone tactile
node tests/e2e/single-file.test.mjs            # fabrique le fichier unique et l'ouvre en file:// : activités, invitation, page « Mise en place », mobile
YP_INDEX=/dist/index.html node tests/e2e/app.test.mjs     # mêmes parcours, mais sur le fichier unique (idem glyphs, cloud)
```

## Ce qui a été testé… et ce qui ne l'a pas été

**Testé automatiquement** (Chromium headless avec caméra/micro factices, plusieurs pages = plusieurs amis, base locale de démo) :
parcours complet du site (inscription, invité, party, invitation, chat, stickers, réactions, vocal, lecteur synchronisé,
réglages, langues), les 7 jeux (parties complètes, spectateurs, déconnexion/reprise, rechargement, FR/EN), le navigateur
cloud (plusieurs participants, SSRF, authentification contre un **faux** serveur Supabase conforme à l'API, ordinateur et
téléphone tactile), l'affichage mobile à 390 px (et de 320 à 700 px pour la barre du haut), l'absence d'emoji, l'absence d'erreur JavaScript. **Le fichier unique** : fabriqué dans un dossier vide puis ouvert en
`file://` (aucun autre fichier demandé, accueil, inscription, party, les 7 activités, lien d'invitation, page « Mise en place » :
copie, repli quand la copie est bloquée, vérification simulée, français, mobile) ; les parcours complets, l'absence d'emoji et le
navigateur cloud ont aussi été rejoués sur lui.

**Non testé ici — à vérifier chez vous au premier lancement :**

- **Le vrai projet Supabase** (inscription, RLS, Realtime privé, RPC) : l'environnement de développement ne pouvait pas
  l'atteindre. Le code d'accès est le même que celui de la démo, mais seule votre première inscription le confirmera.
- Les **migrations 2, 3 et 4** (non appliquées, voir plus haut ; leur SQL n'a donc jamais tourné sur une vraie base PostgreSQL. Il est rejouable, et un
  script collé d'un seul bloc s'exécute en une seule transaction : en cas d'erreur rien n'est appliqué, corrigez la ligne indiquée et relancez) et le chargement depuis les **CDN** (supabase-js, hls.js, API YouTube, polices).
- La **voix et le partage d'écran** entre de vraies machines sur Internet (NAT, TURN), sur de **vrais téléphones**, et
  l'usage multi-appareils.
- Le fichier unique n'a été essayé que dans **Chromium** (Firefox et Safari en `file://` : non testés) ; ouvert depuis le disque,
  YouTube peut refuser de s'afficher (il exige une vraie adresse https), et l'inscription réelle depuis un fichier local n'a pas
  été essayée : hébergé en https, tout cela revient à la normale.
- La page « Mise en place » n'a jamais parlé au vrai projet : sa vérification est testée contre un serveur simulé.
- Le **déploiement du serveur cloud** derrière HTTPS et l'authentification avec un **vrai** jeton Supabase.

**Limites connues du navigateur partagé cloud :** pas de son venant de ce navigateur (utilisez le lecteur synchronisé ou le
partage d'écran avec audio pour un film), pas de contenu protégé DRM (Netflix, Disney+… ne se lisent pas dans Chromium
headless), les participants partagent les mêmes connexions pendant la session. Détails dans `server/README.md`.

## Dépannage

| Symptôme | Cause probable et remède |
|---|---|
| Bandeau « Mode démo : tout reste sur cet appareil » alors que vous voulez Supabase | Le projet ne répond pas. Les projets Supabase **gratuits sont mis en pause après une période sans activité** : Dashboard → votre projet → **Restore**, attendez quelques minutes, rechargez. (Le projet livré était en pause le jour de la livraison ; je l'ai réveillé.) |
| Les amis n'arrivent pas à s'inscrire / pas d'e-mail reçu | Désactivez *Confirm email* (Authentication → Providers → Email) ou configurez votre propre SMTP : le SMTP par défaut de Supabase est très limité. |
| « Continuer comme invité » échoue | Activez *Allow anonymous sign-ins* (Authentication → Sign In / Providers). |
| Les liens magiques ou la confirmation renvoient vers une mauvaise adresse | Authentication → URL Configuration : *Site URL* et *Redirect URLs*. |
| Le micro ou le partage d'écran ne démarrent pas | Il faut **HTTPS** (ou `localhost`) et l'autorisation du navigateur. Sur iPhone, le partage d'écran n'existe pas dans Safari. |
| Deux amis ne s'entendent pas | Réseau strict (4G, entreprise) : ajoutez un serveur **TURN** dans `iceServers` (`js/config.js`). |
| « Navigateur partagé » affiche une page d'installation | Normal tant que `cloud.url` est vide dans `js/config.js` : voir `server/README.md`. |
| Rien ne s'affiche en ouvrant `index.html` par double-clic | Vous avez ouvert le `index.html` du **dossier source** : ses modules ES ne se chargent pas depuis `file://`. Ouvrez le **fichier unique** (`dist/index.html`, ou celui livré à part), ou lancez un serveur HTTP (`python3 -m http.server`). |
| Le lien d'invitation affiché ne marche pas chez mes amis | Le site est ouvert depuis un fichier local : mettez `index.html` en ligne (Netlify, etc.) et ouvrez la party depuis cette adresse. |
| La page « Mise en place » affiche « Non vérifiable » | Le projet Supabase ne répond pas (voir la 1re ligne de ce tableau) ou vous n'êtes pas connecté. |

## Sécurité en deux mots

Clé publique uniquement dans le site ; accès aux données par règles RLS ; canaux temps réel privés réservés aux membres ;
secrets de jeu envoyés à un seul destinataire ; texte des utilisateurs jamais interprété comme du HTML ; serveur cloud avec
authentification, protection contre l'accès au réseau interne (SSRF), contexte isolé par party et limites d'usage.
