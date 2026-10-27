# Cas pratique : analyse des ventes de l'enseigne « Boutique Formation »

## Contexte

Vous venez d'être recruté·e comme **Data Analyst** dans une enseigne française de vente multicanal (site web, application mobile et 10 magasins physiques).
Elle vend de l'informatique, de la téléphonie, des articles pour la maison, du sport et des livres.

La directrice générale prépare le **comité de direction de janvier 2026**. Elle vous demande un bilan complet des ventes **2024-2025** et des recommandations concrètes pour 2026.

Toutes les données sont dans la base MySQL `formation_python_data` (script `data/formation_ventes.sql`) :

| Table | Description |
|---|---|
| `categories` | Les 5 catégories de produits |
| `produits` | Catalogue : prix de vente, coût d'achat, stock actuel |
| `magasins` | Les 10 magasins : ville, région, date d'ouverture, surface |
| `clients` | Fichier clients : identité, sexe, date de naissance, ville, segment (Particulier / Professionnel) |
| `commandes` | En-têtes de commande : client, date, canal, mode de paiement, statut |
| `lignes_commande` | Détail : produit, quantité, prix unitaire, remise (0.10 = 10 %) |

> **Règles de gestion fixées par la direction financière**
> - Le **chiffre d'affaires (CA)** d'une ligne est égal à `quantite × prix_unitaire × (1 − remise)`.
> - Les commandes au statut **« Annulée »** sont **exclues du CA**. Les autres statuts (Livrée, Expédiée, En cours) comptent.
> - La **marge brute** d'une ligne est égale à `CA − quantite × cout_achat`.
> - La date de référence de l'analyse est le **1er janvier 2026**.

Livrable attendu : un notebook Jupyter propre, commenté, avec des graphiques lisibles et une **conclusion rédigée** à chaque partie.

---

## Partie 1 : Connexion et chargement des données

1. **Connexion.** Connectez-vous à la base `formation_python_data` depuis Python (SQLAlchemy + PyMySQL), puis chargez les 6 tables dans 6 DataFrames pandas.
2. **Vérification du chargement.** Affichez le nombre de lignes de chaque table. Les comptes doivent correspondre à ceux de la requête de vérification à la fin du script SQL.

## Partie 2 : Découverte et qualité des données

*Le service informatique vous prévient que le fichier clients « n'est pas parfait ». Avant de présenter un chiffre, vous devez savoir sur quoi vous travaillez.*

3. Pour chaque table, affichez les **types des colonnes**. Les dates sont-elles bien reconnues comme des dates ? Corrigez si nécessaire.
4. Combien de **valeurs manquantes** y a-t-il par colonne et par table ? Lesquelles sont normales (règle métier) et lesquelles sont de vrais défauts de qualité ?
5. Y a-t-il des **doublons** (lignes identiques, ou même email pour plusieurs clients) ?
6. **Contrôles de cohérence** :
   - Toutes les commandes `Magasin` ont-elles un `id_magasin`, et les commandes Web / Application mobile n'en ont-elles pas ?
   - Le `prix_unitaire` des lignes de commande est-il toujours égal au prix du catalogue ?
   - Existe-t-il des commandes passées **avant** la date d'inscription du client ?
   - Existe-t-il des clients **sans aucune commande** ?
7. Rédigez une **note de qualité** de 5 lignes maximum à destination du service informatique.

## Partie 3 : Préparation des données

8. Construisez une **table d'analyse unique** `ventes` (une ligne par ligne de commande) qui contient toutes les informations utiles : commande, client, produit, catégorie, magasin.
9. Ajoutez les colonnes calculées : `ca`, `cout`, `marge`, `taux_marge`, `annee`, `mois` (période AAAA-MM), `jour_semaine`.
10. Calculez l'**âge** de chaque client à la date de référence, puis une **tranche d'âge** : 18-24, 25-34, 35-44, 45-54, 55-64, 65 et plus. Que faites-vous des clients sans date de naissance ?
11. Créez la table `ventes_valides` qui exclut les commandes annulées. C'est elle que vous utiliserez pour tous les calculs de CA.

## Partie 4 : Performance globale

*La directrice veut « les grands chiffres » en première page de la présentation.*

12. Calculez les **KPI** pour 2024 et 2025 : CA, marge brute, taux de marge, nombre de commandes, nombre de clients actifs, **panier moyen** (CA / commande) et nombre moyen d'articles par commande. Calculez aussi l'**évolution en %** entre les deux années.
13. Tracez l'**évolution mensuelle du CA** sur 24 mois. Voyez-vous une **saisonnalité** ? Quels sont les 3 meilleurs et les 3 pires mois ?
14. Quel est le **jour de la semaine** le plus rentable ? Faut-il renforcer les équipes en magasin certains jours ?

## Partie 5 : Produits et catégories

*Le directeur des achats doit renégocier les contrats fournisseurs et réduire le catalogue.*

15. Donnez le **CA, la marge et le taux de marge par catégorie**. Quelle catégorie fait le plus de volume ? Laquelle est la plus rentable ?
16. Établissez le **Top 10 et le Flop 5 des produits** selon le CA. Le classement change-t-il si on trie par marge ?
17. **Loi de Pareto** : quel pourcentage des produits réalise 80 % du CA ? Tracez la courbe de Pareto.
18. **Gestion des stocks.** Calculez les ventes mensuelles moyennes (en quantité) de chaque produit sur 2025 et la **couverture de stock** en mois (`stock / ventes mensuelles moyennes`). Quels produits risquent une **rupture** en moins d'un mois ? Lesquels sont en **surstock** (plus de 12 mois de couverture) ?

## Partie 6 : Clients

*Le service marketing prépare les campagnes 2026 et veut mieux connaître la clientèle.*

19. Combien de commandes un client passe-t-il en moyenne ? Tracez la **distribution** du nombre de commandes par client.
20. Qui sont les **10 meilleurs clients** (CA) ? Quelle part du CA total représentent-ils ?
21. Comparez **Particuliers et Professionnels** : nombre de clients, CA, panier moyen, catégories préférées.
22. Analysez le CA et le panier moyen par **tranche d'âge** et par **sexe**. Quelle catégorie est la plus achetée par chaque tranche d'âge ?
23. **Clients à risque.** Listez les clients dont la dernière commande date de **plus de 6 mois** (par rapport au 01/01/2026). Combien sont-ils et quel CA ont-ils généré par le passé ? C'est la cible de la campagne de réactivation.
24. **Segmentation RFM.** Calculez pour chaque client :
    - **R**écence : le nombre de jours depuis sa dernière commande ;
    - **F**réquence : son nombre de commandes ;
    - **M**ontant : son CA total.

    Attribuez un score de 1 à 4 à chaque dimension (quartiles), puis définissez des segments : *Champions*, *Fidèles*, *À risque*, *Perdus*… Combien de clients et quel CA par segment ?
25. **Analyse de cohortes.** Regroupez les clients selon le **mois de leur première commande**. Calculez le pourcentage de clients de chaque cohorte qui recommandent les mois suivants, puis affichez une **heatmap de rétention**.

## Partie 7 : Canaux et magasins

*Le directeur des opérations envisage de fermer un magasin et d'investir dans l'application mobile.*

26. Calculez la **répartition du CA par canal** (Web, Magasin, Application mobile) et son évolution entre 2024 et 2025. Quel canal progresse le plus ?
27. Le **panier moyen** est-il différent selon le canal ?
28. **Classement des magasins** : CA, nombre de commandes, panier moyen et **CA par m²**. Quel magasin est le moins performant ? Feriez-vous la même recommandation selon que l'on regarde le CA brut ou le CA par m² ?
29. Calculez le **CA par région** à partir des magasins.
30. Quelle est la répartition des **modes de paiement** selon le canal ? Les **espèces** sont-elles utilisées en ligne ? Est-ce normal ?

## Partie 8 : Remises et annulations

*Le contrôleur de gestion pense que « les remises coûtent trop cher ».*

31. Quel montant total de **remises** a été accordé en 2025 ? Quelle part de la marge potentielle cela représente-t-il ?
32. Les lignes avec remise ont-elles une **quantité moyenne** plus élevée que celles sans remise ? La remise est-elle efficace ?
33. Quel est le **taux d'annulation** global, par canal et par catégorie ? Quel **CA a été perdu** à cause des annulations ?

## Partie 9 : Synthèse

34. Construisez un **tableau de bord** d'une page, avec 4 à 6 graphiques, qui résume l'activité.
35. Rédigez **5 recommandations** chiffrées pour le comité de direction (produits, clients, canaux, magasins, remises).

---

### Conseils
- Vérifiez toujours vos totaux : la somme des CA par catégorie doit être égale au CA total.
- Un graphique doit avoir un titre, des axes nommés et des unités (€, %).
- Chaque chiffre présenté doit répondre à une question métier. Concluez chaque partie par une ou deux phrases en français.
