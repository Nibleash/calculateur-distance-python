# Calculateur de Distance à Pied 🚶

Outil web moderne pour enrichir des fichiers Excel/CSV avec les distances et durées de marche entre deux adresses, en utilisant l'API Mapbox.

**Déploié sur GitHub Pages** → [Visitez l'application](#déploiement)

---

## 📋 Logique du Projet

### Architecture Générale

Le projet repose sur deux composants principaux :

1. **Module Python (`distance_calculator.py`)** : Logique métier en pur Python 3, indépendant du contexte d'exécution
2. **Interface Web (`index.html`)** : Application Pyodide + JavaScript côté navigateur

### Double Appel API Mapbox

L'enrichissement suit ce workflow :

```
Fichier Excel/CSV
    ↓
[1] GEOCODING API ──→ Récupère coordonnées (lon, lat) pour chaque adresse unique
    ↓
[2] DISTANCE MATRIX API ──→ Calcule distances/durées entre paires de coordonnées
    ↓
Excel enrichi (4 colonnes ajoutées)
```

#### Phase 1️⃣ : Géocodage (Mapbox Geocoding API)

**Endpoint:** `https://api.mapbox.com/search/geocode/v6/forward`

Pour chaque adresse unique du fichier :
```
GET https://api.mapbox.com/search/geocode/v6/forward?q={adresse}&access_token={token}
```

**Réponse:**
```json
{
  "features": [
    {
      "geometry": {
        "coordinates": [2.3522, 48.8566]  // [longitude, latitude]
      }
    }
  ]
}
```

📊 **Optimisation :** Les adresses sont dédupliquées avant géocodage pour minimiser les appels API.

#### Phase 2️⃣ : Calcul de la Matrice de Distance (Distance Matrix API)

**Endpoint:** `https://api.mapbox.com/directions-matrix/v1/mapbox/walking/`

Avec toutes les coordonnées valides :
```
GET https://api.mapbox.com/directions-matrix/v1/mapbox/walking/{coords}?
  access_token={token}&
  annotations=distance,duration
```

**Format des coordonnées:** `lon1,lat1;lon2,lat2;lon3,lat3;...`

**Réponse:**
```json
{
  "distances": [
    [0, 2541, 3200],      // distances en mètres
    [2541, 0, 1850],
    [3200, 1850, 0]
  ]
}
```

#### Phase 3️⃣ : Enrichissement du DataFrame

Pour chaque ligne du fichier original :
- Récupère les indices des adresses départ/arrivée dans la matrice
- Extrait `distances[idx_from][idx_to]`
- Convertit : distances en m
- Ajoute 4 colonnes au DataFrame :
  - `Adresse entreprise coords` : tuple (lon, lat)
  - `Adresse employé coords` : tuple (lon, lat)
  - `Distance trajet (m)` : float arrondi à 2 décimales

---

## 🛠️ Architecture Technique

### Stack

| Composant | Technologie | Usage |
|-----------|------------|-------|
| **Backend (calculs)** | Python 3 (Pyodide) | Géocodage, calculs, enrichissement |
| **API Externe** | Mapbox (Geocoding + Distance Matrix) | Données géolocalisation & distances |
| **Frontend** | HTML5 + CSS3 + Vanilla JS | Interface utilisateur |
| **Runtime** | WebAssembly (Pyodide) | Exécution Python côté navigateur |
| **Dépendances Python** | pandas, openpyxl, requests | Lecture/écriture Excel, appels HTTP |

### Flux de Données

```
utilisateur
    ↓
[UI] Saisit token Mapbox + upload Excel
    ↓
[JS] Initialise Pyodide → charge packages Python
    ↓
[Python/Pyodide] Exécute enrich_excel_bytes()
    ├── _geocode_one() pour chaque adresse unique
    │   └── pyfetch() → Mapbox Geocoding API
    ├── _matrix() avec toutes les coordonnées
    │   └── pyfetch() → Mapbox Distance Matrix API
    └── Enrichit DataFrame avec résultats
    ↓
[Python] Exporte DataFrame → BytesIO Excel (.xlsx)
    ↓
[JS] Convertit bytes → Blob → URL.createObjectURL()
    ↓
[UI] Affiche lien download
    ↓
utilisateur télécharge fichier enrichi
```

## 🚀 Déploiement

### GitHub Pages

#### Prérequis
- Repository GitHub public
- Fichier `index.html` à la racine (ou dossier `/docs`)

#### Configuration GitHub

1. **Paramètres du repository** → **Pages**
2. Source : `Deploy from a branch`
3. Branch : `main` (ou `gh-pages`)
4. Dossier : `/ (root)` ou `/docs`
5. **Save**

L'application sera accessible à :
```
https://{username}.github.io/{repo-name}
```

✅ **Avantages :**
- Gratuit, hébergé sur GitHub
- Déploiement automatique à chaque push
- Pas de server backend requis
- HTTPS automatique

## 🔑 Clé API Mapbox

1. **Créer un compte** → https://account.mapbox.com
2. **Plan gratuit** : 50 000 requêtes/mois
3. **Copier la clé publique** (commence par `pk.`)
4. **Coller dans l'interface** (reste côté navigateur)

⚠️ **Sécurité** : La clé n'est pas envoyée au serveur, elle reste en mémoire du navigateur.

---

## 💡 Exemples d'Utilisation

### Via Interface Web

1. Ouvrir `index.html` (GitHub Pages ou local)
2. Saisir clé Mapbox
3. Upload Excel (2 colonnes d'adresses)
4. Cliquer "Enrichir"
5. Télécharger le fichier enrichi

### Format Attendu

| Colonne A | Colonne B |
|-----------|-----------|
| 16 Rue Richer, 75009 Paris | 103 Rue du Faubourg Saint-Denis, 75010 Paris |
| 4 Sq. Rapp, 75007 Paris | 101 Bd Raspail, 75006 Paris |

### Format Généré

| Colonne A | Colonne B | Entreprise coords | Adresse employé coords | Distance trajet (km) |
|-----------|-----------|-------------------|------------------------|----------------------|
| ... | ... | (2.343, 48.874) | (2.356, 48.875) | 1.08 |
| ... | ... | (2.301, 48.859) | (2.329, 48.846) | 2.94 |

---

## 📝 Licence

MIT - Libre d'utilisation et de modification

---

## 🐛 Dépannage

### Erreur : "Clé API Mapbox manquante"
→ Vérifier que la clé est bien collée dans le champ

### Erreur : "Aucune adresse géocodée"
→ Vérifier le format des adresses (ex: "rue, code postal, ville")

### Fichier ne se télécharge pas
→ Ouvrir la console (F12) et vérifier les erreurs JavaScript

### Délai long de traitement
→ Normal pour un gros fichier (chaque adresse requiert une requête API)
→ Vérifier que vous n'avez pas atteint la limite Mapbox

---

**Créé avec ❤️ pour Île-de-France Mobilités**
