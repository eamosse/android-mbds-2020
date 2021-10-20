# Gérer la persistance avec Room 
Dans ce TP, nous allons améliorer l'application Entre Voisin développée dans les TPs 2 et 3. 

Jusqu'ici les données de l'application étaient gérées en mémoire, par conséquent quand on relance l'application les données sont réinitialisées. 

Nous allons utiliser une base de données pour gérer la persistance afin que les données persistent au redemarrage de l'application. 

## Partie 0: Créer une nouvelle branche
Ajouter une nouvelle branche (appelez-le room) dans le reposiroty des TPs précédents.

## Partie 1: Mise en place de l'architecture du code 
Le schéma suivant représente l'architecture du code de l'application 
![Architecture](/architecture.png "Nouveau voisin")

### Prenons le temps de comprendre l'architecture du projet
La communinauté android a adopté dans les récentes années le pattern MVVM (Model View View Model). Cette approche permet une meilleur séparation entre les différentes couches de l'application : vues, modèles métiers et accès aux données. 

Le schéma ci-dessus représente le schéma global des projets développés selon le pattern MVVM. 

**UI Controller** : composant d'architecture qui regroupe les vues de l'application. Il comprend généralement les activités et les fragments. 

**ViewModel** : Composant d'architecture qui regroupe les composants qui gère les données des vues. En Android, un viewmodel joue le rôle degestionnaire de données pour les vues. Cela dit, à partir de maintenant, je ne veux plus voir de logique métier dans les fragments et les activités. Pendant le cycle de vie d'une application, les fragments et activités peuvent être détruits puis recréés par le système. Ainsi pour éviter de perdre les données déjà chargées dans la vue, suite à une rotation de l'écran par exemple, on utilise les viewmodels qui ne se détruisent pas quand les composants de vues sont recréées.

**Repository** : Contrairement aux composants de vues et aux viewmodels, les repositories ne sont pas des composants du SDK. Ce sont des classes utilitaires utilisées pour gérer les sources de données d'une application. Ainsi, le repository fait le pont entre les données de l'application et les viewmodels. Le repository garantit une certaine cohérence au niveau des données traitées par les viewmodels et les activités. 

**Data Source** : Composant d'architecture destinée à la gestion des données dans une application Android. Une application Android peut utiliser plusieurs sources : bases de données locales, web services etc. 

Ainsi, dans ce TP la source de données est une base de données SQLLite gérée avec l'ORM Room. Bien sure dans le TP suivant nous allons utiliser un service web. 

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

Room est un ORM (Object Relational Mapper) qui permet de manipuler les tables d'une base de données SQLite en utilisant uniquement des classes, interfaces et des méthodes. Pour plus d'informations, consultez la documentation ici https://developer.android.com/training/data-storage/room. 

Composants Room : 
**Entity** : Classe permettant de modéliser un objet métier, cette classe sera représentée dans la base de données comme une table. 
Concrètement : 
- Le nom de l'entité représente le nom de la classe
- Les attributs de l'entité représentent les colonnes de la table 
- Les instances de l'entité représentent les lignes de la table. 

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
> Resynchronise le projet, gradle téléchargera automatiquement les dépendances

### 2. Ajouter une nouvelle entité

- Dans le package ```room```, créer un nouveau package ```entities``` et ajouter une classe ```NeighborEntity```

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
- address est un val car on ne veut pas qu'on puisse modifier l'adresse d'un voisin; c'est bête mais vous avez compris l'idée
- website peut être null, tous les voisins ne sont pas des geeks. 
- favorite est une variable non obligatoire, car elle vaut false par défaut et elle est modifiable car on peut ajouter ou enlever un voisin en favori à volonté


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

- Une base de données relationnelle peut contenir plusieurs tables, par conséquent l'attribut entities de l'annotation permet de définir les entités gérées par la base de données. 

- Le numéro de version est très important car il permet d'indiquer à Room quand le schéma de la BDD change. Ainsi, à chaque modification du modèle, il faut incrémenter le numéro de version. 

- La classe qui modélise la base de données doit être abstraite, Room générera l'implémentation. 

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

> Petite analyse 

- Cette classe implémente ```NeighborDatasource``` qui lui force à avoir le même comportement que la classe ```InMemoryNeighborDataSource```. Ainsi, la seule différence entre les deux sources de données est le mode de persistance. 

- Ici on a utilisé un ```MediatorLiveData``` qui nous permet d'observer les changements sur la base de données et appliquer des modifications sur les données renvoyées par la BDD avant de les afficher à l'utilisateur. 

#### Convertion de données 
A ce stade, la base de données à son modèle de données qui est différente de celle de la vue. Plus tard, quand on va rajouter des web services, eux aussi vont avoir leur propre modèle de données. Un des principes du clean architecture consiste à utiliser un modèle dédié pour chaque couche et de définir des routines de conversion quand il faut faire passer des données d'une couche à l'autre. 

- Dans le package ```dal```, ajouter un nouveau package ```utils``` et dans ce nouveau package ajouter un fichier Kotlin ```NeighborMapper.kt```. 

- Créer une fonction d'extension dans ```NeighborMapper.kt``` permettant de convertir une instance de ```NeighborEntity``` en ```Neighbor```

> Rappelez-vous, les fonctions d'extension permettent d'implémenter une fonction dans une classe sans la surcharger. 

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

### 6. Adapter le repository 
La data source ```RoomNeighborDataSource``` a besoin d'une instance de l'application pour s'initialiser; en fait c'est la base de données qui en a besoin. 
Pour cela, nous allons modifier le repository pour qu'elle accepte en paramètre une instance d'application. 

> Ce n'est pas très propre de faire cela, on pourrait faire mieux en utilisant des libraires d'injection de dépendances comme Hilt (https://developer.android.com/training/dependency-injection/hilt-android). Pour les plus curieux, ce sera votre point bonus. 

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

- On veut maitriser la création du repositoty, on met son constructeur en private.

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

### Ajouter des données dans la BDD
Afin d'éviter d'avoir une base de données vide, nous allons modifier la classe qui gère la base de données pour y insérer des données de tests à la création de la base de données. 

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
> Compilez et testez

## Partie 3: Injection de dépendance
Notre repository fonctionne mais le processus permettant de récupérer son instance est un peu lourd; on va l'améliorer en utilisant l'injection de dépendance. 

- Sous le package principal du projet, créer un nouveau package ```di``` (pour dependency injection) et dans ce package ajouter une classe ```DI.kt```
```Kotlin
object DI {
    lateinit var repository: NeighborRepository
    fun inject(application: Application) {
        repository = NeighborRepository.getInstance(application)
    }
}
```

> Il existe des librairies qui permettent de gérer l'injection de dépendance plus proprement 
- Hilt (https://developer.android.com/training/dependency-injection/hilt-android)
- Koin (https://insert-koin.io/)

- Modifier la méthode onCreate de l'activity ```MainActivity``` pour instancier le repository once for all :) 
```Kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        DI.inject(application)
        ...
    }
```

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
