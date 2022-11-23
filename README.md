# arangoDB
## Implémentation 
### installation sur Linux

```
curl -OL https://download.arangodb.com/arangodb310/DEBIAN/Release.key 
```
```
sudo apt-key add - < Release.key 
```
```
echo 'deb https://download.arangodb.com/arangodb310/DEBIAN/ /' | sudo tee /etc/apt/sources.list.d/arangodb.list 
```
``` 
sudo apt-get install apt-transport-https
```
``` 
sudo apt-get update 
```
```
sudo apt-get install arangodb3=3.10.1-1
```

Après l’installation, vous pouvez l’utiliser via sa console, une sorte de terminal commande, ou via son interface Web
graphique

Pour allumer le server il faut taper la commande suivant :
```
sudo systemctl start arangodb3
```
Pour voir le Status de Serve
```
sudo systemctl status arangodb3
```
Pour éteindre le server
```
sudo systemctl Stope arangodb3
```
### Arangodb shell
```
arangosh
```
Afficher touts les Bases des données :
```
db._databases();
```
Création de database :
```
db._createDatabase("exempleDB");
```
Pour l'utiliser :
```
db._useDatabase("exempleDB");
```
Supprimer la base de données 
```
db._dropDatabase("mydb");
```
Afficher les Collection
```
db._collections();
```
Création de collections et les afficher
```
db._create('places');
```
Création de documents et les afficher
```
db.places.save({ _key : "foo", city : "foo-city" });
```
remplir la collection par des donnes aléatoire
```
for (var i = 0; i <= 10; i++) {
db.places.save({ _key: "example" + i, zipcode: i })
};
```
Afficher la collection
```
db.places.toArray();
```
Compter les documents
```
db.places.count();
```
Modification de documents
```
db._update("places/foo", { zipcode: 39535 });
db.places.document("foo");
```
Suppression de documents
```
db.places.count();
db._remove("places/example7");
db.places.remove("example8");
db.places.count();
```
Recherche de documents
```
#rechercher par code postal
db.places.byExample({zipcode:3 }).toArray();
#rechercher par code postal et nome de place 
db.places.byExample({_key : "example2" ,zipcode:1 }).toArray();
```
Exécution de requêtes AQL
> Grouper Les Villes  leurs code postale superieur a 5 
```
db._query('
  FOR p IN places
    FILTER p.zipcode >= 5
    RETURN { placeName: p._key, zipcode: p.zipcode}
    ').toArray();
```
    
 Supprimer tout les  collections
 ```
 for (let col of db._collections()) {
    if (!col.properties().isSystem) {
        db._drop(col._name);
    }
}
```
### Interface web
Création des collections
Il existe 2 types de collections dans ArangoDB :
Document  , Edge (dédiée aux graphes).

les documents dans une collection Edge permettent de faire le lien entre deux documents classiques.

Pour le réseau social, on va créer 3 collections du type Document : users ,posts, cities ;
et 1 collection du type Edge pour les commentaires qui fera le lien entre les users et les posts et les comments.

Cities
> On insère 3 villes avec leurs coordonnées GPS.
```
LET cities_data = [
   { _key: "lyon", name: "Lyon",  coordinates: [45.735773, 4.815310] },
   { _key: "paris", name: "Paris",  coordinates: [48.875421, 2.288823] },
   { _key: "bordeaux", name: "Bordeaux",  coordinates: [44.844301, -0.576779] }
]

FOR city IN cities_data INSERT city IN cities
```
Users
> On insère 4 utilisateurs possédant un nom d’utilisateur, un âge et l’id d’une ville précédemment insérée dans la collection cities.
```
LET users_data = [
    { _key: "user_1", username : "user 1", age: 25, city: "cities/lyon" },
    { _key: "user_2", username : "user 2", age: 35, city: "cities/lyon" },
    { _key: "user_3", username : "user 3", age: 55, city: "cities/paris" },
    { _key: "user_4", username : "user 4", age: 60, city: "cities/bordeaux" }
]

FOR user IN users_data INSERT user IN users
```
Posts
> On insère 4 posts avec leur titre et leur contenu.
```
LET posts_data = [
    { _key: "post_1", title: "post 1", content: "content 1" },
    { _key: "post_2", title: "post 2", content: "content 2" },
    { _key: "post_3", title: "post 3", content: "content 3" },
    { _key: "post_4", title: "post 4", content: "content 4" }
]

FOR post IN posts_data INSERT post IN posts
```
Comments
> On insère des commentaires qui font le lien entre des utilisateurs (_from) et des posts (_to).
```
LET comments_data = [
    { _from: "users/user_1", _to: "posts/post_1", text: "Comment of user 1 on post 1" },
    { _from: "users/user_1", _to: "posts/post_3", text: "Comment of user 1 on post 3" },
    { _from: "users/user_2", _to: "posts/post_1", text: "Comment of user 2 on post 1" },
    { _from: "users/user_2", _to: "posts/post_2", text: "Comment of user 2 on post 2" },
    { _from: "users/user_2", _to: "posts/post_3", text: "Comment of user 2 on post 3" },
    { _from: "users/user_3", _to: "posts/post_3", text: "Comment of user 3 on post 3" },
    { _from: "users/user_3", _to: "posts/post_4", text: "Comment of user 3 on post 4" }
]

FOR comment IN comments_data INSERT comment IN comments
```
Les requêtes en AQL de type graphe utilisent le vocabulaire des graphes. 
> obtenir les commentaires de l’utilisateur 2
```
FOR vertex, edge, path IN 1..1 OUTBOUND
  'users/user_2' GRAPH 'users_posts_comments'
  RETURN {
    post_title: vertex.title,
    comment: edge.text
    }
```
> les utilisateurs qui ont commenté le post 3.
```
FOR vertex, edge, path IN 1..1 INBOUND
  'posts/post_3' GRAPH 'users_posts_comments'
  RETURN vertex
```






