# Jeux de données – Formation Python pour la Data Science

Thème commun : une enseigne fictive de vente multicanal (web, magasin, appli mobile), 10 magasins en France, années 2024-2025.
Les identifiants (`id_client`, `id_produit`, `id_commande`, `id_magasin`) sont partagés entre les fichiers : on peut faire des jointures.

| Fichier | Format | Contenu |
|---|---|---|
| `formation_ventes.sql` | Script MySQL | Base `formation_ventes` : categories (5), produits (27), magasins (10), clients (300), commandes (2 000), lignes_commande (~3 900) + vue `v_ventes` |
| `avis_clients.csv` | CSV, séparateur `;`, UTF-8 | 812 avis clients (note 1-5, recommande, commentaire) |
| `ressources_humaines.xlsx` | Excel, 3 feuilles | `Employes` (120), `Objectifs_2025` (CA objectif mensuel par magasin), `Magasins` |
| `livraisons.txt` | Fichier plat à largeur fixe | ~1 000 livraisons des commandes web/mobile |

## Charger la base MySQL
```bash
mysql -u root -p < formation_ventes.sql
```
⚠️ Le script fait `DROP DATABASE IF EXISTS formation_ventes` puis recrée tout.

```python
from sqlalchemy import create_engine   # pip install sqlalchemy pymysql
engine = create_engine("mysql+pymysql://root:MOT_DE_PASSE@localhost/formation_ventes")
df_ventes = pd.read_sql("SELECT * FROM v_ventes", engine)
```

## Lire les autres fichiers avec pandas
```python
import pandas as pd

avis = pd.read_csv("data/avis_clients.csv", sep=";", parse_dates=["date_avis"], dayfirst=True)

employes   = pd.read_excel("data/ressources_humaines.xlsx", sheet_name="Employes")
objectifs  = pd.read_excel("data/ressources_humaines.xlsx", sheet_name="Objectifs_2025")
toutes     = pd.read_excel("data/ressources_humaines.xlsx", sheet_name=None)  # dict de toutes les feuilles

livraisons = pd.read_fwf(
    "data/livraisons.txt",
    widths=[8, 8, 15, 10, 10, 7, 8, 12],
    names=["id_livraison", "id_commande", "transporteur", "date_expedition",
           "date_livraison", "poids_kg", "cout_livraison", "statut"],
    dtype={"date_expedition": str, "date_livraison": str},
)
```

### Structure du fichier plat `livraisons.txt`
| Colonne | Positions | Largeur | Remarque |
|---|---|---|---|
| id_livraison | 1-8 | 8 | |
| id_commande | 9-16 | 8 | → `commandes.id_commande` |
| transporteur | 17-31 | 15 | |
| date_expedition | 32-41 | 10 | format `AAAAMMJJ` |
| date_livraison | 42-51 | 10 | vide si `EN_TRANSIT` |
| poids_kg | 52-58 | 7 | |
| cout_livraison | 59-66 | 8 | en € |
| statut | 67-78 | 12 | LIVRE / RETARD / EN_TRANSIT |

## Pièges volontaires (pour l'exercice de nettoyage)
- **avis_clients.csv** : 12 lignes en doublon, notes et commentaires manquants, `recommande` écrit `Oui`/`oui`/`OUI`, dates au format `JJ/MM/AAAA`.
- **ressources_humaines.xlsx** : 3 salaires manquants, 1 salaire aberrant (salaire annuel saisi au lieu du mensuel), ancienneté à calculer depuis `date_embauche`.
- **Base MySQL** : emails et dates de naissance manquants (`NULL`) chez certains clients, `id_magasin` NULL pour les commandes hors magasin, commandes `Annulée` à exclure du CA.
- **livraisons.txt** : dates en `AAAAMMJJ` à convertir, dates de livraison vides.

## Idées d'analyses
- CA par mois / catégorie / canal / région (`v_ventes`), marge avec `cout_achat`
- Panier moyen, top clients, segmentation RFM
- Délai moyen de livraison par transporteur (jointure livraisons ↔ commandes)
- Note moyenne par produit / catégorie (jointure avis ↔ produits)
- CA réalisé vs `Objectifs_2025` par magasin ; masse salariale par magasin
