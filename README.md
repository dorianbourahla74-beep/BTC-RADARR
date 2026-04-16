# 📡 BTC Institutional Radar

Dashboard web qui agrège les données de positionnement institutionnel sur le Bitcoin et calcule un score composite quotidien : **LONG / NEUTRE / SHORT**.

## 🎯 Ce que ça fait

- Récupère automatiquement chaque jour à 08:00 UTC :
  - **COT Report** (CFTC) — positions hedge funds sur CME BTC futures
  - **Open Interest** (Binance) — variation 7j croisée avec direction du prix
  - **Funding Rates** (Binance Perps) — indicateur contrarian de sentiment
  - **Flux ETF spot** (Farside) — entrées/sorties nets des ETF BTC US
- Calcule un score composite pondéré de **-100 à +100**
- Affiche tout ça dans un dashboard web moderne, accessible depuis ton navigateur

## 🚀 Guide de déploiement (pas-à-pas, zéro expérience requise)

### Étape 1 — Créer un compte GitHub

1. Va sur [github.com](https://github.com) → **Sign up**
2. Choisis un username (ex: `ton-pseudo`), email, mot de passe
3. Valide ton email

### Étape 2 — Créer le repository

1. Une fois connecté, clique sur le **+** en haut à droite → **New repository**
2. Nom du repo : `btc-radar` (ou ce que tu veux)
3. Coche **Public** (obligatoire pour GitHub Pages gratuit)
4. **Ne coche rien d'autre** (pas de README, pas de .gitignore)
5. Clique **Create repository**

### Étape 3 — Uploader les fichiers

**Option A — Via le web (plus simple si tu débutes)**

1. Sur la page de ton repo fraîchement créé, clique **"uploading an existing file"**
2. Glisse-dépose TOUS les fichiers et dossiers de ce projet :
   - `index.html`
   - dossier `scripts/`
   - dossier `data/`
   - dossier `.github/` (important ! celui avec workflows/update.yml)
3. Scroll en bas, écris un message de commit ("Initial setup")
4. Clique **Commit changes**

**Option B — Via Git (si tu connais)**

```bash
git clone https://github.com/TON-PSEUDO/btc-radar.git
cd btc-radar
# copie tous les fichiers ici
git add .
git commit -m "Initial setup"
git push
```

### Étape 4 — Activer GitHub Pages

1. Dans ton repo, va sur **Settings** (onglet en haut à droite)
2. Menu de gauche → **Pages**
3. Sous **"Build and deployment"** :
   - **Source** : `Deploy from a branch`
   - **Branch** : `main` · `/root`
4. Clique **Save**
5. Attends 1-2 minutes, ton site sera disponible à :
   `https://TON-PSEUDO.github.io/btc-radar/`

### Étape 5 — Activer les Actions (automatisation)

1. Va dans l'onglet **Actions** de ton repo
2. Si un message demande d'activer les workflows, clique **"I understand my workflows, go ahead and enable them"**
3. Dans la liste des workflows à gauche, clique sur **"Update BTC Institutional Data"**
4. Clique sur le bouton **"Run workflow"** → **Run workflow** (pour tester immédiatement)
5. Attends ~1 minute, tu verras une ✅ verte apparaître
6. Rafraîchis ton site → les données réelles sont là !

À partir de maintenant, le workflow tourne **automatiquement tous les jours à 08:00 UTC**.

### Étape 6 — Autoriser les push automatiques (IMPORTANT)

Pour que le bot GitHub Actions puisse mettre à jour les données :

1. **Settings** → **Actions** → **General**
2. Scroll jusqu'à **"Workflow permissions"**
3. Sélectionne **"Read and write permissions"**
4. Clique **Save**

## 🔧 Personnalisation

### Changer les pondérations

Ouvre `scripts/fetch_data.py`, cherche la ligne :

```python
WEIGHTS = {"cot": 35, "oi": 20, "funding": 25, "etf": 20}
```

Modifie les valeurs (total doit faire 100). Par exemple, si tu veux donner plus de poids aux flux ETF :

```python
WEIGHTS = {"cot": 25, "oi": 15, "funding": 20, "etf": 40}
```

### Changer les seuils LONG/SHORT

Toujours dans `scripts/fetch_data.py` :

```python
LONG_THRESHOLD = 30     # plus strict = 40 ou 50
SHORT_THRESHOLD = -30   # plus strict = -40 ou -50
```

### Changer la fréquence de mise à jour

Dans `.github/workflows/update.yml`, modifie la ligne `cron`. Quelques exemples :

- `'0 8 * * *'` → tous les jours à 08:00 UTC (actuel)
- `'0 8 * * 1-5'` → lundi à vendredi seulement
- `'0 */6 * * *'` → toutes les 6 heures
- `'0 8 * * 6'` → samedi uniquement à 08:00 UTC

Aide syntaxe cron : [crontab.guru](https://crontab.guru)

## 📊 Comment lire le signal

| Score | Signal | Interprétation |
|-------|--------|----------------|
| ≥ +30 | 🟢 **LONG** | Les institutions accumulent (majorité des indicateurs pointe vers une demande forte) |
| -30 à +30 | 🟡 **NEUTRE** | Positionnement indécis ou signaux contradictoires |
| ≤ -30 | 🔴 **SHORT** | Les institutions distribuent ou se couvrent |

**Attention** : le score n'est pas un signal de trading direct. Les institutions peuvent être longues alors que le prix baisse à court terme. C'est un **indicateur de contexte macro**.

## 🐛 Dépannage

**Le workflow échoue avec une erreur "Permission denied"**
→ Vérifie l'étape 6 : Settings → Actions → General → Workflow permissions → Read and write

**Le site affiche "Impossible de charger les données"**
→ Le workflow n'a pas encore tourné. Va dans Actions et lance-le manuellement.

**Une source affiche "Indisponible"**
→ Normal occasionnellement (Binance peut bloquer certaines IPs, Farside peut changer son HTML). Le score composite s'adapte et ignore les sources indisponibles.

**Les flux ETF sont à zéro le weekend**
→ Normal, les ETF ne tradent que les jours ouvrés US.

## 📝 Sources des données

- **CFTC COT Report** : [publicreporting.cftc.gov](https://publicreporting.cftc.gov) (API publique Socrata)
- **Binance Futures** : [fapi.binance.com](https://fapi.binance.com) (API publique)
- **Farside ETF flows** : [farside.co.uk](https://farside.co.uk/bitcoin-etf-flow-all-data/) (scraping HTML)
- **CoinGecko** : [api.coingecko.com](https://api.coingecko.com) (API publique)

## ⚠️ Disclaimer

Cet outil est à des fins **éducatives et informatives** uniquement. Ce n'est pas un conseil en investissement. Les données peuvent être incomplètes, retardées, ou erronées. Trade à tes risques et périls.
