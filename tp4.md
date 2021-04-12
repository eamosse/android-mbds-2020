# Gérer la persistance avec Room 
Dans ce TP, nous allons améliorer l'application Entre Voisin développée dans les TPs 2 et 3. 

Jusqu'ici les données de l'application étaient gérées en mémoire, par conséquent quand on relance l'application les données sont réinitialisées. 

Nous allons utiliser une base de données pour gérer la persistance afin que les données persistent au redemarrage de l'application. 

## Partie 0: Créer une nouvelle branche dans le TP3
Ajouter une branche room dans le reposiroty du TP 3.

## Partie 1: Mise en place de l'architecture du code 
Le schéma suivant représente l'architecture du code de l'applization 
![Ajoutez un voisin](/architecture.png "Nouveau voisin")

### Prenons le temps de comprendre l'architecture du projet
La communinauté android a adopté dans les récentes années le pattern MVVM (Model View View Model). Cette approche permet une meilleur séparation entre les différentes couches de l'application : graphiques, modèles métiers et accès aux données. 

Le schéma ci-dessus représente le schéma global des projets développés selon le pattern MVVM. 

**UI Controller** : composant d'architecture qui regroupe les vues du module. Il comprend généralement les activités et les fragments. 

**ViewModel** : Composant d'architecture qui regroupe les view models. En Android, un viewmodel est un composant du SDK permettant de gérer les données de la vue. Pendant le cycle de vie d'une application, les fragments et activités peuvent être détruits puis recréés par le système. Ainsi pour éviter de perdre les données déjà chargées dans la vue, suite à une rotation de l'écran par exemple, on utilise les viewmodels qui ne se détruisent pas quand les composants de vues sont recréées.

**Repository** : Contrairement aux composants de vues et aux viewmodels, les repositories ne sont pas des composants du SDK. Ce sont des classes utilitaires utilisées pour gérer les différentes sources de données d'une application. Ainsi, le repository fait le pont entre les données de l'application et les viewmodels.

**Data Source** : Composant d'architecture destinée à la gestion des données dans une application Android. Une application Android peut utiliser plusieurs sources : bases de données locales, web services etc. 
Ainsi, dans le schéma d'architecture la source de données est une base de données SQLLite sur le téléphone. 

### Réorganisation du code de l'application
Nous allons réorganiser le code de l'application pour qu'il réflète mieux l'architecture du projet tel que défini ci-dessus. 

Sous le package principale de l'application : 
- Créer un package ```ui``` puis déplacer l'activity ```MainActivity``` et le package ```fragments``` ainsi que les fichiers qui'il contient dans ce nouveau package; 

- Créer un package ```dal```, puis dans ce nouveau package ajouter deux sous-packages ```room``` et ```memory```. 

- Déplacer la classe ```InMemoryNeighborDataSource``` dans le package ```memory```

- Déplacer la classe ```NeighborDatasource``` dans le package ```dal```

- Renommer le package ```data``` en ```repositories``` 

A ce stade, les packages du projet devraient être organisés comme sur la figure suivante : 

![Ajoutez un voisin](/packages.png "Nouveau voisin")

> Assurez-vous que le projet compile

## Partie 2: Mise en place de la base de données avec Room

Room est une ORM (Object Relational Mapper) qui permet de manipuler les tables d'une base de données SQLite en utilisant uniquement des classes, interfaces et des méthodes. Room fait partie de la suite Jetpacks. 
Pour plus d'informations consulté la documentation ici https://developer.android.com/training/data-storage/room. 

Composants de Room : 
**Entity** : Classe permettant de modéliser un objet métier qui sera représenté dans la base de données comme une table. 
Concrètement : 
- Le nom de l'entité représente le nom de la classe
- Les attributs de l'entité représentent les colonnes de la table 
- Les instances de l'entité représente les lignes de la table. 

Il existe autant d'entités que de tables dans la base de données. 

**Database** : Classe qui modélise la base de données sous forme d'objets. 

**DAO** : Interface permettant de modéliser les opérations à effectuer sur les entités. Il existe généralement autant de DAO que d'entités.

### 1. Dépendances 
Modifier le fichier ```build.gradle``` du module de l'application.

```gradle
  ....
  apply plugin: 'kotlin-kapt'
  ....

  def room_version = "2.2.6"

  implementation "androidx.room:room-runtime:$room_version"
  kapt "androidx.room:room-compiler:$room_version"

```
> Resync le projet, gradle téléchargera automatiquement les dépendances

### 2. Ajouter une nouvelle entité

- Dans le package ```room```, créer un nouveau package ```entities```. Ajouter une classe ```NeighborEntity``` dans le package ```entities```

```Kotlin
@Entity(tableName = "neighbors")
class NeighborEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long,
    var name: String,
    var avatarUrl: String,
    val address: String,
    var phoneNumber: String,
    var aboutMe: String,
    var favorite: Boolean = false,
    var webSite: String? = null,
)
```
> Observez bien les variables: 
- id est un val et c'est la clef primaire
- address est un val car on ne veut pas qu'on puisse modifier l'adresse d'un voisin; c'est bête mais vous comprenez l'idée
- website peut être null, tous les voisins ne sont pas des geeks. 
- favorite est une variable non obligatoire, car elle vaut false par défaut


### 3. Ajouter une DAO 
- Dans le package ```room```, créer un sous-package ```daos```. Ajouter une interface ```NeighborDao``` dans le package ```daos```.

> Be careful, quand vous définissez le package ```daos```, assurez-vous de ne pas l'ajouter dans ```entities```. 

- Modifiez l'interface ```NeighborDao``` pour y ajouter une méthode pour récupérer la liste des neighbors dans la base de données 

```Kotlin
@Dao
interface NeighborDao {
    @Query("SELECT * from neighbors")
    fun getNeighbors(): LiveData<List<NeighborEntity>>
}
```
> Ici nous avons défini une méthode permettant de récupérer la liste des voisins. Cette liste est retournée comme un liveData; ainsi l'inteface de l'application qui ajoutera les voisins sera notifiée quand les données de l'entities change dans la base de données.

### 4. Modéliser la base de données Room

- Dans le package ```room```, ajouter une classe ```NeighborDataBase``` permettant de modéliser la base de données. 

```Kotlin
@Database(
    entities = [NeighborEntity::class],
    version = 1
)
abstract class NeighborDataBase : RoomDatabase() {
    abstract fun neighborDao(): NeighborDao

    companion object {
        private var instance: NeighborDataBase? = null
        fun getDataBase(application: Application): NeighborDataBase {
            if (instance == null) {
                instance = Room.databaseBuilder(
                    application.applicationContext,
                    NeighborDataBase::class.java,
                    "neighbor_database.db"
                )
                    .fallbackToDestructiveMigration()
                    .build()
            }
            return instance!!
        }
    }
}

```

> Pour mieux comprendre : 
- L'annotation ```@Database``` permet de définir les entités et la version de la base de données. 

- Une base de données relationnelle peut contenir plusieurs tables, par conséquent l'attribut entities de l'annotation permet de définir plusieurs entités. 

- Le numéro de version est très important car il permet d'indiquer à Room quand le schéma de la BDD change. Ainsi, à chaque modification du modèle, il faut incrémenter le numéro de version. 

- La classe qui modélise la base de données doit être abstraite, c'est room qui générera l'implémentation. 

- Les DAOs sont référencés comme des méthodes abstraites dans la classe modélisant la base de données. Les implémentations seront générées par Room. 

- Finalement, la base de données doit être un singleton. Cette approche est importante car elle permet d'avoir un seul point d'entrée sur la base de données.

- fallbackToDestructiveMigration() indique à Room de supprimer la base de données et la recréer quand le modèle de données change. 

### 5. Implémenter une nouvelle source de données 
Dans cette section, nous allons définir une classe qui permet d'exposer les données de la base de données selon le protocole défini par l'interface ```NeighborDatasource```.

- Dans le package ```room```, ajouter une classe ```RoomNeighborDataSource``` qui implémente l'interface ```NeighborDatasource```

```Kotlin

class RoomNeighborDataSource(application: Application) : NeighborDatasource {
    private val database: NeighborDataBase = NeighborDataBase.getDataBase(application)
    private val dao: NeighborDao = database.neighborDao()

    private val _neighors = MediatorLiveData<List<Neighbor>>()

    init {
        _neighors.addSource(dao.getNeighbors()) { entities ->
            _neighors.value = entities.map { entity ->
                entity.toNeighbor()
            }
        }
    }

    override val neighbours: LiveData<List<Neighbor>>
        get() = _neighors

    override fun deleteNeighbour(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }

    override fun createNeighbour(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }

    override fun updateFavoriteStatus(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }

    override fun updateNeighbour(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }
}

```

- Dans le package ```dal```, ajouter un nouveau package ```utilis``` et dans ce nouveau package ajouter un fichier Kotlin ```NeighborMapper.kt```. 

- Créer une fonction d'extension dans ```NeighborMapper.kt``` permettant de convertir une instance de ```NeighborEntity``` en ```Neighbor```

> Les fonctions d'extension permettent d'implémenter une fonction dans une classe sans la surcharger. 

```Kotlin
fun NeighborEntity.toNeighbor() = Neighbor(
    id = id,
    name = name,
    avatarUrl = avatarUrl,
    address = address,
    phoneNumber = phoneNumber,
    aboutMe = aboutMe,
    favorite = favorite,
    webSite = webSite ?: ""
)
```

> Petite analyse 

- Cette classe implémente ```NeighborDatasource``` qui lui force à avoir le même comportement que la classe ```InMemoryNeighborDataSource```. Ainsi, la seule différence entre les deux sources de données est le mode de persistance. 

- Ici on a utilisé un ```MediatorLiveData``` qui nous permet d'observer les changements sur la base de données et appliquer des modifications sur les données renvoyées par la BDD avant de les afficher à l'utilisateur. 

Cette étape est importante car la base de données retourne une liste de ```NeighborEntity``` alors que la vue attend une liste de ```Neighbor```. 

Là encore on est face à un principe de clean architecture. La base de données à son modèle données qui est différente de celle de la vue. 


Pour faciliter la communication entre la couche de données et la vue, il faut créer une routine permettant de convertir les données au bon format attendu par la couche de destination. 

Ici, on veut renvoyer des données de la base de données à la vue, on a besoin d'une méthode qui convertit une entité en modèle métier. Pour cela l'instance du ```MediatorLiveData``` observe les changements sur les données de la BDD et quand elles changent on applique une conversion sur les données avant de mettre à jour le live data. 

### 6. Adapter le repository 
La data source ```RoomNeighborDataSource``` a besoin d'une instance de l'application pour s'initialiser; en fait c'est pas la base de données qui en a besoin. 

Pour cela, nous allons modifier le repository pour qu'elle accepte en paramètre une instance d'application. 

### 1. Modifier le repository

```Kotlin
class NeighborRepository private constructor(application: Application) {
    private val dataSource: NeighborDatasource

    init {
        dataSource = RoomNeighborDataSource(application)
    }

    // Méthode qui retourne la liste des voisins
    fun getNeighbours(): LiveData<List<Neighbor>> = dataSource.neighbours

    fun delete(neighbor: Neighbor) = dataSource.deleteNeighbour(neighbor)

    companion object {
        private var instance: NeighborRepository? = null

        // On crée un méthode qui retourne l'instance courante du repository si elle existe ou en crée une nouvelle sinon
        fun getInstance(application: Application): NeighborRepository {
            if (instance == null) {
                instance = NeighborRepository(application)
            }
            return instance!!
        }
    }
}
```

> On essaie de comprendre ? 

- On veut maitriser l'instanciation du repositoty, on met son constructeur en private.

- On veut que le reposirory soit un singleton, on crée une méthode statique pour l'initialiser et on rend son constructeur private. Ainsi, on ne pourra pas l'instancier à lextérieur de la classe. 

- Finalement, on change de source de données; on remplace ```InMemoryNeighborDataSource``` par ```RoomNeighborDataSource``` 

### 2. Fixer les erreurs de compilation
L'instanciation du repository nécessite une instance de application en paramètre.

- Modifier la méthode setData pour récupérer une instance de l'application et la passer en paramètre au repository. 
```Kotlin
private fun setData() {
        // Récupérer l'instance de l'application, si elle est null arrêter l'exécution de la méthode
        val application: Application = activity?.application ?: return

        val neighbors = NeighborRepository.getInstance(application).getNeighbours()
        neighbors.observe(viewLifecycleOwner) {
            val adapter = ListNeighborsAdapter(it, this)
            binding.neighborsList.adapter = adapter
        }
    }
```   

> Essayez de compiler et tester. Que remarquez-vous ? Pourquoi ? 

- Vous devriez avoir une page blanche, ce qui est logique car l'a base de données est vide. 

### Ajouter des données dans la BDD
- Ajouter un callback lors de la l'instanciation de la BDD pour intercepter l'événement de création de la BDD. 
```Kotlin
fun getDataBase(application: Application): NeighborDataBase {
            if (instance == null) {
                instance = Room.databaseBuilder(
                    application.applicationContext,
                    NeighborDataBase::class.java,
                    "neighbor_database.db"
                ).addCallback(object : Callback() {
                    override fun onCreate(db: SupportSQLiteDatabase) {
                        super.onCreate(db)
                        insertFakeData()
                    }
                })
                    .fallbackToDestructiveMigration()
                    .build()
            }
            return instance!!
        }

        private fun insertFakeData() {
            Executors.newSingleThreadExecutor().execute {
                InMemory_NeighborS.forEach {
                    instance?.neighborDao()?.add(it.toEntity())
                }
            }
        }
```
> Nous avons ajouté un callback lors de l'instanciation de la base de données pour intercepté la création de la base de données. 

> Compilez et testez

## Partie 3: Injection de dépendance
Notre repository fonctionne mais le processus permettant de récupérer son instance est un peu lourd; on va l'améliorer en utilisant l' injection de dépendance. 

- Sous le package principal du projet, créer un nouveau package ```di``` (pour dependency injection) et dans ce package ajouter une classe ```DI.kt```
```Kotlin
object DI {
    lateinit var repository: NeighborRepository
    fun inject(application: Application) {
        repository = NeighborRepository.getInstance(application)
    }
}
```

- Modifier la méthode onCreate de l'activity ```MainActivity``` pour instancier le repository once for all :) 
```Kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        DI.inject(application)
        ...
    }
```

> Pour récupérer l'instance du repository dans le ViewModel il suffit d'appeler ```DI.repository``` 

- Remplacer les références à ```Repository.getInstance()``` par ```DI.repository``` 


## Partie 4: Utiliser un View Model 
Notre architecture n'est pas totalement clean, car le fragment appel directement le repository.

On va utiliser un View Model pour gérer les données du fragment et la communication avec le repository. 

- Ajouter les dépencances aux librairies viewModel et liveData
```gradle
def lifecycle_version = "2.3.0"
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
```

- Sous le package principal du projet, ajouter un package ```viewmodels``` et une nouvelle classe ```RepositoryViewModel``` dans le package ```viewmodels```. 

- Modifier la classe ```RepositoryViewModel``` pour qu'elle implémente ViewModel. 

```Kotlin
class NeighborViewModel : ViewModel() {
    private val repository: NeighborRepository = DI.repository

    // On fait passe plat sur le résultat retourné par le repository
    val neighbors: LiveData<List<Neighbor>>
        get() = repository.getNeighbours()
}
```

- Modifier le fragment ```ListNeighborsFragment``` pour récupérer la liste des voisins via le viewmodel ```NeighborViewModel```. 

1. Instancier le viewModle 
```Kotlin
***
    private lateinit var viewModel: NeighborViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel = ViewModelProvider(this).get(NeighborViewModel::class.java)
    }
***
```
> Avez-vous remarqué ? 
- On ne crée pas une instance de viewModel mais on la récupére dans un provider ```ViewModelProvider```. L'appel à la méthode ```get``` vérifie si une instance du viewmodel existe, si oui il le retourne; sinon il crée une nouvelle instance, l'ajouter dans le provider et le retourner.

- Un ```ViewModelProvider``` est lié soit à une activité ou un fragment. Ici, ```ViewModelProvider(this)``` fait référence au gestionnaire de viewmodel du fragment. Ainsi, tant que le fragment existe, le viewmodel existe aussi; quand le fragment est détruit, le viewmodel est détruit ainsi que les données qu'il contient.

- Si on veut partager un viewmodel entre deux fragments, il faut le le lier à l'activité et non à chaque fragment. 

2. Observer le livedata exposant la liste des voisins dans le fragment

```Kotlin
private fun setData() {
        viewModel.neighbors.observe(viewLifecycleOwner) {
            val adapter = ListNeighborsAdapter(it, this)
            binding.neighborsList.adapter = adapter
        }
    }

``` 

## Partie 5: A vous de jouer
Maintenant vous allez coder les amis :) 

1. Ajouter les méthodes pour supprimer un voisin quand on clique sur l'icone suppression. 

> ATTENTION, vous n'avez pas besoin de raffraichir la liste, elle sera mise à jour automatiquement via le live Data. 

2. Ajouter les méthodes pour ajouter un nouveau voisin dans la base de données à partir du formulaire créé dans le TP 3. 

3. Ajouter un menu dans la toolbar (en haut à droite) permettant à l'utilisateur de choisir la source de données à utiliser : 
- Non persistent --> InMemoryDataSource
- Persistent --> RoomDatasource

4. Ajouter une vue pour afficher le détail d'un voisin. 

5. Ajouter un bouton dans la vue de détail permettant d'ajouter un voisi en favori. 
