# Lago — dossier de reprise

*Consolidé le 3 août 2026, à partir des trois sessions « Lago v4 super-app features »
(20 juillet → 30 juillet 2026) et des mémoires du projet.*

---

## 1. Ce qu'est Lago

Application de VTC pour **Abidjan**, prototypée en **fichier HTML unique** avec le framework
maison **X-DC** (balise `<x-dc>`, directives `sc-if` / `sc-for` / `{{ binding }}`, classe
`Component extends DCLogic`, méthode `renderVals()`).

Identité visuelle à conserver : `#0A2F35`, `#0FA3A8`, `#17B9AE`, glassmorphism,
**carte SVG d'Abidjan dessinée à la main** (lagune Ébrié, quartiers, ponts, bateaux animés
— 394 éléments).

---

## 2. Le cahier des charges v4 d'origine (ton prompt fondateur)

### A — Abonnements & Mode Travailleur
- **4 formules** : Navetteur 32 000 F · Étudiant 18 000 F · Premium 75 000 F · Famille 45 000 F,
  chacune avec quota, distance max, classe autorisée et conditions d'annulation.
- **Mode Travailleur** : planning hebdomadaire (cases L-MM-J-V + heure de départ, défaut 7 h),
  jauge de crédits, annulation sans pénalité avant 22 h la veille, bouton « Pause pour ce mois ».

### B — Les 5 piliers futuristes
| Pilier | Contenu |
|---|---|
| **Lago Dispatch** | Minibus partagé, lignes fixes en heure de pointe (Yopougon → Plateau), 500 F, réservé aux abonnés |
| **Awa Vision** | Caméra de bord opt-in, détection nids-de-poule / freinages, rapport « Conduite sereine » /10 |
| **Bouclier tarifaire** | Prix figé 6 mois pour Premium/Navetteur, badge « Prix garanti » |
| **Team Lago** | Comptes entreprise, invitations par e-mail, plages horaires, facturation centralisée, code promo perso |
| **Lago Slot** | Créneaux de 15 min, chauffeur engagé, course offerte si retard ≥ 5 min |

### C — IA
- **Routing prédictif anti-bouchon** : `trafficFactor` selon l'heure (7-9 h = 1,8 · 12-14 h = 1,2),
  au moins **3 itinéraires alternatifs** sur la carte SVG, l'IA choisit le plus rapide.
  Formule affichée en tooltip : `T_est = Σ (dᵢ/vᵢ + λwᵢ)`.
- **Awa négociatrice** : champ « Proposer un prix », acceptation entre **70 % et 90 %** du prix initial.

### D — Télémétrie & sécurité physique
- Accéléromètre + gyroscope simulés, événement « conduite dangereuse » au-delà de 0,8,
  jauge **Score de sérénité** sur l'écran En route.
- Classe **Robust** (4×4 / tout-terrain, +30 % sur le Confort).

### E — Expérience native iOS simulée
- **Dynamic Island / Live Activity** : pilule flottante « Kouamé B. · arrive dans 3 min », cliquable.
- **Widget écran verrouillé** simulé.

### F — Étudiant & communautaire
- **Pass Campus** : 12 000 F/mois, 8 trajets domicile ↔ Université FHB, **carte de la zone 5 km**.
- **Portefeuille partagé (Famille)** : recharger le compte d'un proche via Orange Money.

**Contraintes** : fichier unique, structure X-DC conservée, tout dans `this.state`, navigation
vers les nouveaux écrans, simulation cohérente, identité visuelle respectée, Awa comme centre
névralgique (« Je veux m'abonner », « Je propose 1500 F », « Active Awa Vision »).

---

## 3. Chronologie de ce qui a été construit

1. **Lago v4** — cahier des charges ci-dessus livré en fichier unique.
   Pièges corrigés : `tagName` → `localName` dans le moteur de secours (sinon écran noir),
   masquage du gabarit `<x-dc>` en JS et non en CSS.
2. **Persistance `localStorage`** — ta proposition avait trois défauts (callbacks `setState`
   inexistants, noms de champs faux, `rating`/`tip` à ne pas persister). Refaite via
   `componentDidUpdate`, testée par rechargement réel.
3. **Backend `~/lago-server`** — Express/ESM, 36 routes, 6 domaines, base JSON.
   Bug réel trouvé : le **Bouclier tarifaire ne figeait pas le prix** (il neutralisait le surge
   mais pas la distance, qui bougeait avec le détour anti-bouchon).
4. **Lago v5** — ta v5 ne démarrait pas (crash `S.getLevel(...)`, `this._store = null`,
   12 écrans manquants). Réparée, complétée, animations fluides ajoutées.
   Bug « grosses bordures noires » : la publication supprime le `<head>` → tout style critique
   va dans `<helmet>`.
5. **Coquille iOS `~/Lago-iOS`** — WKWebView + schéma `lago://` (jamais `file://`, sinon
   localStorage bloqué). Sert à **tester et montrer**, pas à publier.
6. **Le tournant (27 juillet)** — tu as demandé ce qui manquait pour un vrai lancement.
   Réponse : tout est simulé, et une WKWebView autour d'un HTML se fait refuser par Apple
   (règle 4.2). Tu as choisi **« un vrai service qui roule »**.
7. **Supabase branché en vrai (28 juillet)** — SQL déployé, boucle de course complète.
8. **Lago v6 Live** — l'app appelle vraiment le serveur.
9. **APK Android (29 juillet)** — Capacitor, chaîne d'outils installée, app installable.
10. **Performance + animation de la voiture (29-30 juillet)** — la partie la plus longue.

---

## 4. Décisions produit figées (ne pas les rouvrir sans raison)

- **Stack** : Capacitor (UI web → iOS + Android) · **Android d'abord** (les chauffeurs
  d'Abidjan y sont) · Supabase (Postgres + auth SMS + temps réel) · **OpenStreetMap / OSRM**
  (pas Google : clé + carte bancaire ; pas Apple : 99 $/an + JWT) · agrégateur de paiement
  ivoirien (CinetPay / PayDunya / Hub2 = Orange Money + MTN + Wave d'un coup) · FCM.
- **MVP** : une seule boucle vraie — vraie carte + GPS → prix réel → chauffeur le plus proche →
  suivi live → **cash d'abord** — sur **une seule zone** (Cocody ↔ Plateau) avec
  **5-10 chauffeurs recrutés à la main**.
- **Reporté en v2** : abonnements / Bouclier tarifaire, négociation Awa, fidélité, minibus,
  télémétrie. Ce sont tes différenciateurs, pas ton MVP.
- **Hors code, lent et critique** : société (CEPICI), licence VTC, assurance transport de
  personnes, comptes marchands, KYC chauffeurs. Le vrai tueur de projets VTC n'est pas
  technique : c'est le **démarrage à deux faces**.
- Plan complet : artefact « Lago — Plan de lancement »
  → https://claude.ai/code/artifact/68ebe844-bb5a-4837-a58d-512e4c689685

---

## 5. Carte du projet sur ton Mac

| Chemin | Rôle |
|---|---|
| `~/Downloads/Lago v6 Live.html` | **La version de référence** (v6 « Confort & IA », 24 écrans) |
| `~/Downloads/Lago v4 Final.html`, `Lago v5 Final.html` | Versions précédentes |
| `~/lago-server/` | Backend Express (36 routes) + pages servies |
| `~/lago-server/supabase/ALL_lago.sql` | **Tout le SQL, idempotent** — se recolle en entier |
| `~/lago-server/lago-app.html`, `lago-driver.html`, `lago-live.html` | Copies servies + téléphone du chauffeur + démo bout-en-bout |
| `~/lago-android/` | Projet Capacitor, `appId` `ci.lago.app`, minSdk 22 |
| `~/lago-android/build-www.py` | **Génère** `www/index.html` — ne jamais l'éditer à la main |
| `~/lago-android/publier.py` | Estampille l'heure, copie vers `lago-server`, régénère le mobile |
| `~/lago-chauffeur-demo/` | Chauffeur de démonstration en Node (`npm start`) |
| `~/Lago-iOS/` | Coquille WKWebView pour le simulateur |
| `~/Downloads/Lago.apk` | APK installable (4 Mo) |

**Commandes**

```bash
cd ~/lago-chauffeur-demo && npm start
```

```bash
python3 ~/lago-android/publier.py && cd ~/lago-android && npm run apk
```

**Accès Supabase** : projet `https://jpgvwltvofnxxyhkzaib.supabase.co`,
clé **publishable** `sb_publishable_JPJg5PTvQTsc98XhJQv4uA_YCwdkmd2` (jamais la `service_role`).
Anonymous sign-ins **activé**. Blocs `01` → `07` déployés en production (branche `main`).
`99_avant_ouverture_publique.sql` **ne doit pas être exécuté maintenant** — il casserait les démos.

---

## 6. Les règles à ne jamais casser

**Carte et mouvement**
- **La carte dessinée n'est pas géoréférencée.** La position vient du **tracé dessiné**,
  l'avancement vient du **vrai chauffeur** (`liveFraction()` projette le GPS sur l'itinéraire
  réel → fraction → `pointAt()` sur le tracé dessiné). Écart mesuré : **0 px**.
- Jamais de repli sur la position GPS projetée ; avancement **monotone** (aucun recul) ;
  position initiale au début du chemin dessiné.
- Ne jamais reprojeter l'itinéraire OSRM pour le **dessiner** sur cette carte.
- Caméra par paliers de **6 px** (`camPas()`, `transform 260ms linear`) — 3,2× moins de
  transformations. Ne pas revenir au suivi au pixel près.
- Chauffeur de démo : un relevé toutes les **~450 ms** ; glissement = **1,25 ×** l'intervalle mesuré.

**Argent**
- Le prix du **serveur** fait foi partout (`fareFinal()` renvoie `_live.priceLock`).
  Bug corrigé : 2 700 F affichés contre 2 100 F en base.
- Bouclier tarifaire : prix calculé sur `routing.reference_distance_km`, **jamais** sur la
  distance de l'itinéraire retenu.
- Le serveur ne fait jamais confiance à un prix envoyé par le client.
- **Coefficient de détour 1,30** — la ligne droite sous-payait le chauffeur d'environ 30 %
  (lagune Ébrié + ponts obligés). Palier temporaire : la vraie solution est une Edge Function
  qui route côté serveur.

**Parcours réel**
- **Aucun chauffeur ≠ course simulée.** Le repli sur la simulation ne vaut que si le serveur
  est injoignable.
- Une course vit **en base**, pas dans l'onglet (`my_active_ride()` au démarrage).
- Quitter l'écran **annule vraiment** côté serveur.
- Resynchronisation toutes les 8 s + au réveil de l'onglet.
- Le passager ne démarre pas la course lui-même.
- Tout appel RPC qui échoue **arrête le flux**.
- `persistSession: true` côté chauffeur, sinon chaque rechargement crée un fantôme.
- **Chauffeur fantôme** : matching restreint aux positions de moins de **90 s**
  (`05_matching_frais.sql`).

**Méthode de travail**
- **Marqueur de version obligatoire** : `window.LAGO_BUILD` (actuellement **`30/07 23h24`**),
  affiché à côté de « Riviera Golf 4 » sur téléphone. Quand un défaut est signalé :
  **demander d'abord ce marqueur** — plusieurs corrections ont été refaites deux fois parce
  qu'une page périmée était filmée.
- **Ne jamais adopter comme base un fichier réécrit ailleurs.** Trois fois, une version collée
  a perdu des écrans (12 en v5, 5 en v6), le moteur X-DC ou tout le pont Supabase. Bonne
  méthode : garder la version qui tourne et y greffer les nouveautés, ancrage par ancrage.
  Mieux : envoyer **seulement la nouveauté**, ou la décrire en un paragraphe.
- **Ne jamais coller du SQL dans un message** — toujours en fichier (le markdown mange les `*`).
- Publication Artifact : tout style critique dans `<helmet><style>`, **jamais** dans `<head>`.
- `requestAnimationFrame` est **gelé** dans un onglet en arrière-plan et dans le navigateur
  d'inspection : vérifier les animations par la logique, pas à l'œil.

---

## 7. Offre d'abonnement actuelle (30 juillet 2026)

**7 formules** : Éducation+ 15 000 · Pass Campus 12 000 · Étudiant+ 22 000 · Navetteur+ 38 000 ·
Couple+ 42 000 · Famille+ 50 000 · Premium+ 85 000 F/mois.
Chacune porte `rides` (quota qui pilote la jauge du Mode Travailleur) et `privileges`
(pastilles sur la carte **et** sur l'accueil).

**Écran Sur mesure** : trajet + jours + trajets/jour → **180 F/km**, remise de volume −10 % à
5 jours, −5 % à 4 jours, borné 10 000–90 000 F. Souscrire crée une vraie entrée `surmesure`
dans `this.PLANS` **à son propre prix** et remet `worker.used` à zéro.

---

## 8. État exact au 30 juillet 2026, 23 h 24

✅ Boucle de course réelle vérifiée de bout en bout contre Supabase :
chauffeur en ligne → matching géoloc → 6,19 km / 2 100 F → `accepted` → `arriving` →
`in_progress` → `completed`, journal `ride_events` complet, note + pourboire enregistrés,
reprise après coupure vérifiée.
✅ Itinéraire routier réel (OSRM) · APK Android installable · 24 écrans · carte intacte.

**Le dernier point resté ouvert** : la fluidité de l'animation de la voiture pendant la
**phase d'approche**, sur ton téléphone. Dernière correction publiée en `30/07 23h24` ;
ton dernier essai filmait une page antérieure, donc **le test avec cette build n'a jamais été
refait**.

### Ce qui reste devant toi

1. **Refaire le test d'approche** avec la build `30/07 23h24` (vérifier le marqueur affiché).
2. **Recoller `ALL_lago.sql`** si ce n'est pas déjà fait depuis le `07` (facturation routière).
3. **L'arbitrage de la carte** : garder la carte dessinée (position approchée, avancement vrai)
   ou passer à une vraie carte OSM (tout coïncide, identité graphique perdue).
4. **Connexion par SMS** (Supabase Auth) à la place des sessions anonymes.
5. **Edge Function de routage** pour que le prix ne vienne jamais du téléphone.
6. **Avant le Play Store** : targetSdk 35, signature release (keystore à conserver
   précieusement), compte développeur 25 $.
