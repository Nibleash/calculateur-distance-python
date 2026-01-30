# Calculateur de Distance à Pied 🚶

Outil web pour enrichir vos fichiers Excel/CSV avec les distances à pied entre deux adresses, via l'API Mapbox.

**Accès :** Ouvrez `index.html` dans votre navigateur.

---

## 🎯 Comment ça marche ?

### Workflow simplifié

1. **Vous uploadez** un fichier Excel avec 2 colonnes d'adresses
2. **Nous nettoyons** les adresses (suppression apt, escalier, bâtiment, etc.)
3. **Nous géocodons** chaque adresse unique → coordonnées (lon, lat)
4. **Nous calculons** les distances à pied par batches via Mapbox Distance Matrix API
5. **Nous retournons** votre fichier enrichi avec 3 colonnes :
   - `Adresse entreprise coords` → (longitude, latitude)
   - `Adresse employé coords` → (longitude, latitude)
   - `Distance trajet (m)` → distance à pied en mètres

### Format d'entrée attendu

**Fichier Excel/CSV avec 2 colonnes :**

| Colonne 1 (Entreprise) | Colonne 2 (Employé) |
|---|---|
| 16 Rue Richer, 75009 Paris | 103 Rue du Faubourg Saint-Denis, 75010 Paris |
| 4 Sq. Rapp, 75007 Paris | 101 Bd Raspail, 75006 Paris |

---

## 🚀 Utilisation

### Étapes

1. Ouvrir `index.html` dans un navigateur
2. Saisir votre **clé API Mapbox** (voir ci-dessous)
3. Uploader votre fichier Excel/CSV
4. Cliquer sur **"Enrichir"**
5. Télécharger le fichier enrichi

### Clé API Mapbox

**Gratuit :** 100 000 requêtes/mois

1. Créer un compte → https://account.mapbox.com
2. Copier la clé publique (commence par `pk.`)
3. Coller dans le champ de l'application

⚠️ **Sécurité :** La clé reste en mémoire du navigateur, jamais envoyée à un serveur.

---

## 🛠️ Stack technique

- **Frontend** : HTML5, CSS3, JavaScript vanilla
- **Runtime** : Pyodide (Python en WebAssembly)
- **API** : Mapbox Geocoding + Distance Matrix
- **Dépendances** : pandas, openpyxl

---

## 🐛 Dépannage

| Problème | Solution |
|---|---|
| **"Clé API Mapbox manquante"** | Vérifier que la clé est bien collée |
| **Coordonnées nulles** | Format d'adresse invalide ou non reconnue par Mapbox (essayer : "rue, code postal, ville") |
| **Fichier ne se télécharge pas** | Ouvrir F12, vérifier la console pour erreurs JS |
| **Traitement lent** | Normal pour gros fichiers (géocodage séquentiel + appels API par batches de 12) |

---

**Créé avec ❤️ pour Île-de-France Mobilités**
