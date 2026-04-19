# ThermoData Chatbot

Service de chat IA self-hosted (propulsé par OpenAI), avec fallback automatique vers email en cas d'indisponibilité.

## 🏗️ Architecture

```
thermodata.fr                     ← site principal (1 ligne ajoutée)
       │
       │ <script src="https://...netlify.app/chatbot.js">
       ▼
thermodata-chatbot.netlify.app    ← ce service (séparé)
       ├── /chatbot.js            (widget servi au navigateur)
       └── /.netlify/functions/chat   (proxie vers OpenAI)
```

## 📦 Structure

```
thermodata-chatbot/
├── public/
│   ├── chatbot.js          # Widget frontend (JS auto-injecté)
│   └── index.html          # Page de demo
├── netlify/
│   └── functions/
│       └── chat.js         # API proxy vers OpenAI
├── netlify.toml            # Config Netlify
└── README.md
```

## 🚀 Déploiement (3 étapes)

### 1. Crée un repo GitHub

```bash
cd thermodata-chatbot
git init
git add .
git commit -m "init: ThermoData chatbot service"
git remote add origin git@github.com:<ton-user>/thermodata-chatbot.git
git push -u origin main
```

### 2. Déploie sur Netlify

- Va sur https://app.netlify.com → **Add new site** → **Import existing project**
- Sélectionne le repo `thermodata-chatbot`
- Build : tout est dans `netlify.toml`, clique **Deploy**
- Note l'URL générée (ex: `thermodata-chatbot.netlify.app`)

### 3. Configure les variables d'environnement

Dans le dashboard Netlify du site → **Site settings** → **Environment variables** → **Add a variable** :

| Key              | Value                                           |
|------------------|-------------------------------------------------|
| `OPENAI_API_KEY` | `sk-proj-...` (depuis platform.openai.com/api-keys) |

⚠️ **Utilise une NOUVELLE clé** (jamais celle que tu as pu copier-coller ailleurs).

Puis **Trigger redeploy** pour que la variable soit prise en compte.

## ⚙️ Configuration

### A. URL du service dans le widget

Édite `public/chatbot.js` ligne ~13 :

```js
const SERVICE_BASE_URL = 'https://thermodata-chatbot.netlify.app';
//                                  ↑ remplace par ton URL Netlify réelle
```

### B. Domaines autorisés

Édite `netlify/functions/chat.js` ligne ~10 :

```js
const ALLOWED_ORIGINS = [
  'https://thermodata.fr',
  'https://www.thermodata.fr',
];
```

### C. Modèle OpenAI

Édite `netlify/functions/chat.js` ligne ~20 :

```js
const MODEL = 'gpt-4o-mini'; // ou 'gpt-4o', 'gpt-4.1-mini', etc.
```

Commit + push → Netlify redéploie automatiquement.

## 🔌 Intégration sur thermodata.fr

Une seule ligne à ajouter avant `</body>` :

```html
<script src="https://thermodata-chatbot.netlify.app/chatbot.js" defer></script>
</body>
```

## 🛡️ Fallback "plus de crédit"

Le widget bascule automatiquement vers `contact@thermodata.fr` quand l'API renvoie :

| Code HTTP | Cause OpenAI                           |
|-----------|----------------------------------------|
| `401`     | Clé API invalide ou révoquée           |
| `429`     | **Quota / crédit épuisé** (`insufficient_quota`) ou rate limit |
| `5xx`     | API OpenAI en panne                    |

L'utilisateur voit alors un message avec lien `mailto:` pré-rempli.

## 💰 Coût estimé

Avec **gpt-4o-mini** : très économique pour un chat support. Tu peux mettre un **usage limit** sur ton projet OpenAI (dashboard → Settings → Limits) pour plafonner la dépense mensuelle.

## 🔐 Sécurité — règle d'or

La clé API ne doit exister **qu'à un seul endroit** : la variable d'environnement Netlify. Jamais :
- ❌ En dur dans `chat.js`
- ❌ Dans un fichier `.env` commité
- ❌ Partagée par email / chat / Slack

Si une clé fuite par erreur : **révoque-la immédiatement** sur https://platform.openai.com/api-keys et régénères-en une nouvelle.

## 🧪 Test local

```bash
npm install -g netlify-cli
netlify dev
# → ouvre http://localhost:8888
```

N'oublie pas de créer un fichier `.env` local (jamais commité, déjà dans `.gitignore`) :

```
OPENAI_API_KEY=sk-proj-...
```
