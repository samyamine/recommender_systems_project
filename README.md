# Movie Pairing Recommender System Project
### Samy Amine - SCIA 2025

Le présent rapport vise à expliquerla méthodologie que j'ai suivi tout au long du projet, ainsi que mes expérimentations et mes résultats. Nous structurerons cela par tâche.


## Task 1. Collecte et Prétraitement des Données
Pour ce projet, j'ai travaillé avec les datasets MovieLens et IMDb.

Dans le dataset MovieLens j'ai utilisé les fichiers **movies.dat**, **ratings.dat** et **users.dat** afin d'avoir des informatons sur les films, les utilisateurs et les notes que chaque utilisateur a attribué aux films. Pour le dataset IMDb j'ai uniquement utilisé les fichiers **name.basics.tsv** et **title.basics.tsv** afin d'avoir des informations complémentaires sur chaque film ainsi que la liste des personnes ayant contribué à leur réalisation (acteurs, producteurs, etc...).

Dans un premier temps, les datasets issus des fichers **movies.dat** et **title.basics.tsv** ont été merge en fonction du nom et de la date de sortie du film. Cela m'a permis d'obtenir un dataset contenant toutes les informations associées à chaque film, indépendamment de leur pertinence. C'est dans la seconde tâche que je vais m'occuper de trier la donnée.


## Task 2. Feature Engineering
Dans cette seconde tâche, je me suis occupé de trier le dataset préalablement constitué afin de ne garder que la donnée pertinente. L'objectif est d'obtenir un dataset final homogène et prêt à l'emploi pour notre système de recommandation.

Dans un premier temps, j'ai décidé de ne garder que les films et de supprimer tous les autres types de média. Pour cela, j'ai trié la colonne **titleType** du dataset pour ne garder que les lignes dont la valeur est **movie**.

Dans un second temps, j'ai voulu ajouter au dataframe le staff pour chaque film. En effet, les gens peuvent être attirés par un acteur ou par un réalisateur en particulier. C'est pour cela que j'ai trouvé nécessaire de laisser la possibilité aux utilisateurs de connaitre le staff pour chaque film. Pour cela, j'ai ajouté une colonne **cast** indiquant les ids du staff ayant participé à la réalisation de chaque film.

Après avoir effectué cet ajout de feature, j'ai voulu vérifier le bien fondé de conserver l'indication des films pour adulte dans le dataframe. Pour cela, j'ai analysé la répartition des valeurs dans la colonne **isAdult**, et je me suis rendu compte qu'aucun film retenu n'était classé comme appartenant dans la catégorie pour les adultes. J'ai donc fait le choix de supprimer cette feature du dataframe.

Finalement, j'en ai déduit les seules features qui étaient pertinentes pour caractériser chaque film. Ces caractéristiques sont fondées sur les critères de sélection probables des utilisateurs:
- Titre: un utilisateur peut chercher un film particulier par son titre
- Genres: un utilisateur peut avoir des genres préférés
- Année de sortie: un utilisateur peut être intéressé par une année cinématographique particulière. De plus, l'année peut servir à distinguer plusieurs film partageant le même nom.
- Durée du film: certaines personnes préfèrent les films plus longs ou plus courts
- Equipe de production: les utilisateurs peuvent par exemple être attirés par les films du réalisateur James Cameron

Pour représenter les données du couple, j'ai décidé de prendre tous les ratings pour chaque user du couple sous forme de vecteur et de calculer la moyenne de chacune des coordonnées du vecteur.

En conclusion de cette étape, j'ai réalisé la matrice user-item représentant un utilisateur pour chaque ligne et un film pour chaque colonne. Le croisement d'une ligne et d'une colonne contient la note que l'utilisateur a attribué au film. Puisqu'il manque de nombreuses valeurs dans la matrice (sparse matrix), j'ai décidé de remplacer toutes ces valeurs par 0.


## Task 3. Développement du Modèle
Durant la phase de développement du modèle de recommandation, j'ai eu le choix d'implémenter plusieurs algorithmes différents. Parmis eux, j'ai hésité entre le Funk-SVD (Single Value Decomposition) et le ALS (Aleternating Least Squares).

Ces deux algorithmes peuvent répondre à noter besoin. Toutefois, le Funk-SVD réduit la squared error entre la matrice user-items originale et la matrice reconstruite par le biais d'une descente de gradient.
D'autre part, l'ALS cherche à minimiser la même fonction d'optimisation mais mais en utilisant une méthode d'alternance entre la user matrix et la item matrix.

Mon choix final s'est porté sur le ALS. En effet, cet algorithme a pour avantage d'être facilement parallélisé, ce qui en fait un choix d'avenir si mon modèle venait à être utilisé pour un dataset plus volumineux.

Pour l'implémentation, j'ai d'abord normlisé les notes dans l'intervalle [0;1] afin de faciliter la convergence. De plus j'ai utilisé la librairie implicit pour faciliter l'implémentation.
J'ai utilisé les hyperparamètres suivants:
- factors: 64
- regularization: 0.01
- iterations: 50

J'ai ensuite testé le modèle sur le couple constitué des deux utilisateurs ayant noté le plus de films et les résultats étaient cohérents.


## Task 4. Algorithme de Recommandation
Puisqu'il s'agit de faire des recommandations pour un couple, il a été nécessaire de calculer un "score de couple". Pour cela, j'ai décidé de calculer la moyenne des facteurs de chaque membre du couple. Ainsi, en multipliant ce vecteur par les facteurs des films, on se retrouve avec un score qui prend en compte les goûts des deux utilisateurs pour chaque film.


## Task 5. Évaluation
Pour l'évaluation, j'ai décidé de subdiviser le dataset en sets d'entraînement et de test avec la répartition 80%/20%.

Pour évaluer le modèle j'ai choisi d'utiliser la Mean Squared Error (MSE) comme métrique de performance. Ce choix est justifié par le fait que l'ALS cherche à minimiser une fonction objectif, et donc la MSE permet de visualiser précisément à quel point la fonction objectif est minimisée.

Division des données: Les données ont été divisées en ensembles d'entraînement et de test pour évaluer le modèle.
Évaluation du modèle: Le modèle a été évalué en utilisant une métrique choisie (par exemple, la précision, le rappel). Cela permet de mesurer la qualité des recommandations et d'améliorer le modèle si nécessaire.

En effectuant des prédictions sur le test set, j'arrive à une MSE de 0.142


## Conclusion et Interprétation Critique des Résultats
Pour commencer la conclusion, j'aimerais d'abord appuyer sur le fait que le travail mené sur la donnée (travail sur les datasets, feature engineering) a donné de bons résultats. J'ai aussi réussi à utiliser presque toute la donnée disponible. Les films retenus et les features retenues pour chaque film sont pertinents. De plus, la data obtenue est prête a être exploitée.

D'autre part, on arrive bien à créer une user-item interaction matrix avec des données correctes et homogènes.

Enfin, l'erreur obtenue par notre modèle est intéressante. Le modèle a de bonnes performances.

Toutefois, certains points seront à noter. D'abord, les features des films ne sont pas utilisées dans le système de recommendations. C'est dommage car les features de chaque film sont intimement liées aux préférences des utilisateurs et leur exploitation permettrait certainement de développer un modèle beaucoup plus performant.

Enfin, le modèle pourrait être mieux fine-tuné, notamment au niveau de ses hyperparamètres de base (factor et iterations) pour obtenir de meilleures performances.
