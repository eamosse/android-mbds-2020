# Créez une application permettant de mise en relation entre voisins

Dans ce TP, vous allez créer une application permettant de mettre en relation des voisins d'une localité. L'objectif est de créer un ensemble d'interfaces permettant: 
1. D'afficher la liste des voisins 
2. D'ajouter un voisin 
3. D'afficher les détails d'un voisin 
4. De supprimer un voisin

> Dans le cadre de ce TP, la liste des voisins sera gérée en mémoire vive, c'est-à-dire qu'une fois l'application fermée la liste sera réinitialisée. 

**Concepts couverts dans ce TP**
- Gestion de fragments 
- Utilisation de recyclerView et des adapteurs

## Partie 1: Affichage de la liste des voisins

## Configuration du projet
- Créer un un nouveau module dans le projet (le même que le TP précédent) et nommez le **neighbors**
- Choisissez Empty Activity
- Cliquez sur Finish 

## Ajoutez un fragment pour la gestion de la liste des voisins 
> Se référer aux slides du cours pour les notions théoriques sur les fragments 
>> Un fragment est considéré comme une portion de vue que vient s'ajouter dans une activité et qui rempli une fonction bien déterminée. 

>>Ils permettent d'avoir une meilleur séparation de code; l'activité n'a plus de logique applicative mais les délgue aux différents fragments. 

>> L'activité est considérée comme un gestionnaire de fragment et de navigation et la logique applicative est gérée dans les différents fragments. 

- Cliquez sur le nom de package principale du projet et selectionnez **new > package**; dans l'interface qui s'affiche ajoutez fragments. 
> Cela devrait ressembler à ```com.estia.neighbors.fragments```

- Cliquez sur le nouveau package puis **new > Kotlin File/Class**; entrez ListNeighborsFragment et selectionnez Class

- Modifiez la classe **ListNeighborsFragment** pour qu'elle étende la classe Fragment

```kotlin 
    class ListNeighborsFragment : Fragment(){
}
```

- Ajoutez un layout pour gérer la vue du fragment. Cliquez sur **res > layout** puis **new > layout resource file**. Nommez le fichier list_neighbors_fragment puis cliquez sur ok. 

- Dans le .gradle du projet neighbors, ajoutez dans la section dependencies 
```gradle
    implementation "androidx.recyclerview:recyclerview:1.1.0"
```
Cliquez sur sync pour resynchroniser le projet 

> Cette librairie permet d'importer les classes necessaires permettant d'utiliser les RecyclerView 

- Modifiez le layout en y ajoutant un **RecyclerView**
> Un RecyclerView est une vue de groupe permettant d'afficher une collection de vues à partir d'une collection d'objets. Les RecyclerViews utilisent des adapters pour faire la liaison entre la vue et la collection d'objets. 


```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/neighbors_list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:listitem="@layout/neighbor_item" />

</androidx.constraintlayout.widget.ConstraintLayout>

```

> Ajoutez un autre layout **neighbor_item** qui represente la manière dont chaque Neighbor sera affiché dans la liste

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:clickable="true"
    android:focusable="true"
    android:orientation="vertical">


    <ImageView
        android:id="@+id/item_list_avatar"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_marginStart="16dp"
        android:layout_marginTop="10dp"
        android:layout_marginBottom="10dp"
        android:src="@drawable/ic_baseline_person_outline_24"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/item_list_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="24dp"
        android:maxWidth="250dp"
        android:maxLines="1"
        android:textStyle="bold"
        app:layout_constraintBottom_toBottomOf="@+id/item_list_avatar"
        app:layout_constraintStart_toEndOf="@+id/item_list_avatar"
        app:layout_constraintTop_toTopOf="@+id/item_list_avatar"
        tools:text="Mon voisin" />

    <ImageButton
        android:id="@+id/item_list_delete_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="16dp"
        android:background="@null"
        app:layout_constraintBottom_toBottomOf="@+id/item_list_name"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="@+id/item_list_name"
        app:srcCompat="@drawable/ic_baseline_delete_24" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

- Ajoutez deux icones ic_baseline_delete_24 et ic_baseline_person_outline_24
Bouton droit sur le dossier drawable puis selectionnez Vector Assets, ckiquez sur l'icone à droite de clip art et choisissez un icon delete. 
Répétez l'action pour l'icon person. 


- Modifiez la classe ListNeighborsFragment pour y associer la vue qu'on vient de dérinir. Pour cela, surchargez la fonction onCreateView pour qu'elle retourne une instance du layout list_neighbors_fragment comme vue principale.

```kotlin
    /**
     * Fonction permettant de définir une vue à attacher à un fragment 
     */
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.list_neighbors_fragment, container, false)
    }

```

## Modifiez l'activité **MainActivity** pour afficher le fragment 

- Ouvrez le layout de l'activité et modifiez son contenu en y ajoutant un container de fragment 

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

- Ajoutez une méthode dans l'activité permettant de changer de fragment 

```kotlin 
 private fun changeFragment(fragment: Fragment) {
        supportFragmentManager.beginTransaction().apply {
            replace(R.id.fragment_container, fragment)
            addToBackStack(null)
        }.commit()
    }

````

- A la création de l'activité, ajoutez le fragment ListNeighborsFragment comme fragment initial 

```kotlin 
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)
        changeFragment(ListNeighborsFragment())
    }
```

- Exécutez le projet 

> Normalement, vous devez avoir une page blanche. Cela s'explique par le fait qu'il n'y ait pas encore de données dans le fragment ListNeighborsFragment. 
>> Un recycler view est vide par défaut

## Gestion des données du recycler view 

### Commencons par modéliser l'objet principal de notre application (Neighbor)

- Ajoutez un nouveau package ```[package_principal].models```

- Dans ce nouveau package, créez une data classe Neighbor
> En Kotlin, une data class est un type de classe dont le seul but est de modéliser des objets métiers. Elles ne contient que des attributs et n'ont généralement pas de fonctions et n'autorisent pas l'héritage. 

``` kotlin
data class Neighbor(
    val id: Long,
    val name: String,
    val avatarUrl: String,
    val address: String,
    val phoneNumber: String,
    val aboutMe: String,
    val favorite: Boolean,
    val webSite: String
)
```

### Ajoutons un Repository pour gérer toutes les données de l'application 

> En Android, on utilise le terme repository pour caractériser les classes gérant les données de l'application. En procédant ainsi, nous séparons la couche vue, de la couche métier et de gestion de données. 

- Ajoutez un nouveau package ```[package_principal].data```

- Dans ce nouveau package, ajoutez une classe qu'on NeighborRepository

- Dans le package ``data``ajoutez un sous package service 

- Dans le package service ajoutez une interface modélisant toutes les actions qu'il est possible d'effectuer sur un ``Neighbor``


```kotlin
interface NeighborApiService {
    /**
     * Get all my Neighbors
     * @return [List]
     */
    val neighbours: List<Neighbor>

    /**
     * Deletes a neighbor
     * @param neighbor : Neighbor
     */
    fun deleteNeighbour(neighbor: Neighbor)

    /**
     * Create a neighbour
     * @param neighbor: Neighbor
     */
    fun createNeighbour(neighbor: Neighbor)

    /**
     * Update "Favorite status" of an existing Neighbour"
     * @param neighbor: Neighbor
     */
    fun updateFavoriteStatus(neighbor: Neighbor)

    /**
     * Update modified fields of an existing Neighbour"
     * @param neighbor: Neighbor
     */
    fun updateDataNeighbour(neighbor: Neighbor)
}

```

- Toujours dans le package service, ajoutez une classe ```DummyNeighborApiService```
> N'ayant pas de web service, nous allons tout gérer en local pour l'instant 

```kotlin
class DummyNeighborApiService : NeighborApiService {

    override val neighbours: List<Neighbor>
        get() = DUMMY_NeighborS

    override fun deleteNeighbour(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }

    override fun createNeighbour(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }

    override fun updateFavoriteStatus(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }

    override fun updateDataNeighbour(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }


    private val DUMMY_NeighborS: List<Neighbor> = listOf(
        Neighbor(
            1, "Caroline",
            "https://i.picsum.photos/id/1011/5472/3648.jpg?hmac=Koo9845x2akkVzVFX3xxAc9BCkeGYA9VRVfLE4f0Zzk",
            "Saint-Pierre-du-Mont ; 5km",
            "+33 6 86 57 90 14",
            "Lorem ipsum dolor sit amet, consectetur adipiscing elit. " +
                    "Pellentesque porttitor id sem ut blandit. Lorem ipsum dolor sit amet, " +
                    "consectetur adipiscing elit. Donec sed hendrerit ex. Cras a tempus risus. " +
                    "Aliquam egestas nulla non luctus blandit. Aenean dignissim massa ultrices volutpat bibendum. " +
                    "Integer semper diam et lorem iaculis pulvinar.",
            true,
            "www.facebook.fr/caroline"
        ),
        Neighbor(
            2,
            "Jack",
            "https://i.picsum.photos/id/1012/3973/2639.jpg?hmac=s2eybz51lnKy2ZHkE2wsgc6S81fVD1W2NKYOSh8bzDc",
            "Saint-Pierre-du-Mont ; 5km",
            "+33 6 00 55 90 14",
            ("Sed eget fringilla mauris, ac rutrum mauris. Curabitur finibus felis id justo porttitor, " +
                    "vitae hendrerit justo imperdiet. Donec tempus quam vulputate, elementum arcu a, molestie felis. " +
                    "Pellentesque eu risus luctus, tincidunt odio at, volutpat magna. Nam scelerisque vitae est vitae fermentum. " +
                    "Cras suscipit pretium ex, ut condimentum lorem sagittis sit amet. Praesent et gravida diam, at commodo lorem. " +
                    "Praesent tortor dui, fermentum vitae sollicitudin ut, elementum ut magna. Nulla volutpat tincidunt lectus, vel malesuada " +
                    "ante ultrices id.\n\n" +
                    "Ut scelerisque fringilla leo vitae dictum. Nunc suscipit urna tellus, a elementum eros accumsan vitae. " +
                    "Nunc lacinia turpis eu consectetur elementum. Cras scelerisque laoreet mauris ac pretium. Nam pellentesque " +
                    "ut orci ut scelerisque. Aliquam quis metus egestas, viverra neque vel, ornare velit. Nullam lobortis justo et ipsum " +
                    "sodales sodales. Etiam volutpat laoreet tellus, ultrices maximus nulla luctus ultricies. Praesent a dapibus arcu. In at magna " +
                    "in velit placerat vehicula nec in purus."),
            true,
            "www.facebook.fr/jack"
        ),
        Neighbor(
            3,
            "Chloé",
            "https://i.picsum.photos/id/1027/2848/4272.jpg?hmac=EAR-f6uEqI1iZJjB6-NzoZTnmaX0oI0th3z8Y78UpKM",
            "Saint-Pierre-du-Mont ; 6km",
            "+33 6 86 13 12 14",
            "Sed ultricies suscipit semper. Fusce non blandit quam. ",
            false,
            "www.facebook.fr/chloe"
        ),
        Neighbor(
            4,
            "Vincent",
            "https://i.picsum.photos/id/22/4434/3729.jpg?hmac=fjZdkSMZJNFgsoDh8Qo5zdA_nSGUAWvKLyyqmEt2xs0",
            "Saint-Pierre-du-Mont ; 11km",
            "+33 6 10 57 90 19",
            ("Etiam quis neque egestas, consectetur est quis, laoreet augue. " +
                    "Interdum et malesuada fames ac ante ipsum primis in faucibus. Morbi ipsum sem, " +
                    "commodo in nisi maximus, semper dignissim metus. Fusce eget nunc sollicitudin, dignissim " +
                    "tortor quis, consectetur mauris. "),
            true,
            "www.facebook.fr/vincent"
        ),
        Neighbor(
            5,
            "Elodie",
            "https://i.picsum.photos/id/399/2048/1365.jpg?hmac=Tm7jwbWj0i70u952g5yC0da-gxScdY2mQ6gjKrP8Haw",
            "Saint-Pierre-du-Mont ; 8km",
            "+33 6 86 57 90 14",
            ("Fusce in ligula mi. Aliquam in sagittis tellus. Suspendisse tempor, velit et " +
                    "cursus facilisis, eros sapien sollicitudin mauris, et ultrices " +
                    "lectus sapien in neque. "),
            true,
            "www.facebook.fr/elodie"
        ),
        Neighbor(
            6,
            "Sylvain",
            "https://i.picsum.photos/id/375/5184/3456.jpg?hmac=3OUWWnSmq1CUXU7cmTnctSvhQYvyME_osftkbJynX04",
            "Saint-Pierre-du-Mont ; 6km",
            "+33 6 86 12 22 02",
            ("Integer pulvinar lacinia augue, a tempor ex semper eget. Nam lacinia " +
                    "lorem dapibus, pharetra ante in, auctor urna. Praesent sollicitudin metus sit " +
                    "amet nunc lobortis sodales. In eget ante congue, vestibulum leo ac, placerat leo. " +
                    "Nullam arcu purus, cursus a sollicitudin eu, lobortis vitae est. Sed sodales sit " +
                    "amet magna nec finibus. Nulla pellentesque, justo ut bibendum mollis, urna diam " +
                    "hendrerit dolor, non gravida urna quam id eros. "),
            false,
            "www.facebook.fr/sylvain"
        ),
        Neighbor(
            7,
            "Laetitia",
            "https://i.picsum.photos/id/628/2509/1673.jpg?hmac=TUdtbj7l4rQx5WGHuFiV_9ArjkAkt6w2Zx8zz-aFwwY",
            "Saint-Pierre-du-Mont ; 14km",
            "+33 6 96 57 90 01",
            "Vestibulum non leo vel mi ultrices pellentesque. ",
            true,
            "www.facebook.fr/laetitia"
        ),
        Neighbor(
            8,
            "Dan",
            "https://i.picsum.photos/id/453/2048/1365.jpg?hmac=A8uxtdn4Y600Z5b2ngnn9hCXAx8sUnOVzprtDnz6DK8",
            "Saint-Pierre-du-Mont ; 1km",
            "+33 6 86 57 90 14",
            "Cras non dapibus arcu. ",
            true,
            "www.facebook.fr/dan"
        ),
        Neighbor(
            9,
            "Joseph",
            "https://i.picsum.photos/id/473/5616/3744.jpg?hmac=4tP7GutJ3LGRXeprD581uaNnJJGrhF57f08OOtMm1q0",
            "Saint-Pierre-du-Mont ; 2km",
            "+33 6 86 57 90 14",
            ("Donec neque odio, eleifend a luctus ac, tempor non libero. " +
                    "Nunc ullamcorper, erat non viverra feugiat, massa odio facilisis ligula, " +
                    "ut pharetra risus libero eget elit. Nulla malesuada, purus eu malesuada malesuada, est " +
                    "purus ullamcorper ipsum, quis congue velit mi sed lorem."),
            false,
            "www.facebook.fr/joseph"
        ),
        Neighbor(
            10,
            "Emma",
            "https://i.picsum.photos/id/64/4326/2884.jpg?hmac=9_SzX666YRpR_fOyYStXpfSiJ_edO3ghlSRnH2w09Kg",
            "Saint-Pierre-du-Mont ; 1km",
            "+33 6 14 71 70 18",
            ("Aliquam erat volutpat. Mauris at massa nibh. Suspendisse auctor fermentum orci, " +
                    "nec auctor tortor. Ut eleifend euismod turpis vitae tempus. Sed a tincidunt ex. Etiam ut " +
                    "nibh ante. Fusce maximus lorem nibh, at sollicitudin ante ultrices vel. Praesent ac luctus ante, eu " +
                    "lacinia diam. Quisque malesuada vel arcu vitae euismod. Ut egestas mattis euismod.\n\n" +
                    "Vestibulum non leo vel mi ultrices pellentesque. Praesent ornare ligula " +
                    "non elit consectetur, pellentesque fringilla eros placerat. Etiam consequat dui " +
                    "eleifend justo iaculis, ac ullamcorper velit consectetur. Aliquam vitae elit ac ante ultricies aliquet " +
                    "vel at felis. Suspendisse ac placerat odio. Cras non dapibus arcu. Fusce pharetra nisi at orci rhoncus, " +
                    "vitae tristique nibh euismod."),
            false,
            "www.facebook.fr/emma"
        ),
        Neighbor(
            11,
            "Patrick",
            "https://i.picsum.photos/id/91/3504/2336.jpg?hmac=tK6z7RReLgUlCuf4flDKeg57o6CUAbgklgLsGL0UowU",
            "Saint-Pierre-du-Mont ; 5km",
            "+33 6 20 40 60 80",
            ("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce porttitor molestie " +
                    "libero, nec porta lectus posuere a. Curabitur in purus at enim commodo accumsan. Nullam molestie diam a massa finibus, in bibendum " +
                    "nulla ullamcorper. Nunc at enim enim. Maecenas quis posuere nisi. Mauris suscipit cursus velit id condimentum. Ut finibus turpis " +
                    "at nulla finibus sollicitudin. Nunc dolor mauris, blandit id ullamcorper vel, lacinia ac nisi. Vestibulum et sapien tempor, " +
                    "egestas lorem vitae, faucibus urna. Duis id odio massa. Class aptent taciti sociosqu ad litora torquent per conubia nostra, " +
                    "per inceptos himenaeos.\n\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce porttitor molestie  " +
                    "libero, nec porta lectus posuere a. Curabitur in purus at enim commodo accumsan. Nullam molestie diam a massa finibus, in bibendum " +
                    "nulla ullamcorper. Nunc at enim enim. Maecenas quis posuere nisi. Mauris suscipit cursus velit id condimentum. Ut finibus turpis " +
                    "at nulla finibus sollicitudin. Nunc dolor mauris, blandit id ullamcorper vel, lacinia ac nisi. Vestibulum et sapien tempor, " +
                    "egestas lorem vitae, faucibus urna. Duis id odio massa. Class aptent taciti sociosqu ad litora torquent per conubia nostra, " +
                    "per inceptos himenaeos."),
            true,
            "www.facebook.fr/patrick"
        ),
        Neighbor(
            12,
            "Ludovic",
            "https://i.picsum.photos/id/804/6000/3376.jpg?hmac=AZ4VZdij0jPu8BKZRbiE2lEMJGGjSFv43ii3RHRugco",
            "Saint-Pierre-du-Mont ; 5km",
            "+33 6 00 01 10 11",
            ("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce porttitor molestie " +
                    "libero, nec porta lectus posuere a. Curabitur in purus at enim commodo accumsan. Nullam molestie diam a massa finibus, in bibendum " +
                    "nulla ullamcorper. Nunc at enim enim. Maecenas quis posuere nisi. Mauris suscipit cursus velit id condimentum. Ut finibus turpis " +
                    "at nulla finibus sollicitudin. Nunc dolor mauris, blandit id ullamcorper vel, lacinia ac nisi. Vestibulum et sapien tempor, " +
                    "egestas lorem vitae, faucibus urna. Duis id odio massa. Class aptent taciti sociosqu ad litora torquent per conubia nostra, " +
                    "per inceptos himenaeos.\n\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce porttitor molestie  " +
                    "libero, nec porta lectus posuere a. Curabitur in purus at enim commodo accumsan. Nullam molestie diam a massa finibus, in bibendum " +
                    "nulla ullamcorper. Nunc at enim enim. Maecenas quis posuere nisi. Mauris suscipit cursus velit id condimentum. Ut finibus turpis " +
                    "at nulla finibus sollicitudin. Nunc dolor mauris, blandit id ullamcorper vel, lacinia ac nisi. Vestibulum et sapien tempor, " +
                    "egestas lorem vitae, faucibus urna. Duis id odio massa. Class aptent taciti sociosqu ad litora torquent per conubia nostra, " +
                    "per inceptos himenaeos."),
            false,
            "www.facebook.fr/ludovic"
        )
    )


}
```

- Modifiez la classe NeighborRepository
```kotlin 
class NeighborRepository {
    private val apiService: NeighborApiService

    init {
        apiService = DummyNeighborApiService()
    }

    fun getNeighbours(): List<Neighbor> = apiService.neighbours

    companion object {
        private var instance: NeighborRepository? = null
        fun getInstance(): NeighborRepository {
            if (instance == null) {
                instance = NeighborRepository()
            }
            return instance!!
        }
    }
}
````
> On veut que la classe  ```NeighborRepository``` soit un singleton, ainsi une seule instance existera pour toute l'application 

> Aussi, on modifie la classe pour exposer les données à la vue via des fonctions publiques, d'où la fonction ``` getNeighbours ```

### Création d'un adapter pour le recyclerview 

- Ajoutez un nouveau package ``` [package_principale].adapters```

- Dans le package adapter, ajoutez une nouvelle classe ListNeighborsAdapter

- Modifiez la classe pour qu'elle étende la super classe RecyclerView.Adapter 

```kotlin 
package com.mbds.myapplication.adapters

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageButton
import android.widget.ImageView
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.mbds.myapplication.R
import com.mbds.myapplication.models.Neighbor

class ListNeighborsAdapter(
    items: List<Neighbor>
) : RecyclerView.Adapter<ListNeighborsAdapter.ViewHolder>() {
    private val mNeighbours: List<Neighbor> = items
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view: View = LayoutInflater.from(parent.context)
            .inflate(R.layout.neighbor_item, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val neighbour: Neighbor = mNeighbours[position]
        // Display Neighbour Name
        holder.mNeighbourName.text = neighbour.name
    }

    override fun getItemCount(): Int {
        return mNeighbours.size
    }

    class ViewHolder(view: View) :
        RecyclerView.ViewHolder(view) {
        val mNeighbourAvatar: ImageView
        val mNeighbourName: TextView
        val mDeleteButton: ImageButton

        init {
            // Enable click on item
            mNeighbourAvatar = view.findViewById(R.id.item_list_avatar)
            mNeighbourName = view.findViewById(R.id.item_list_name)
            mDeleteButton = view.findViewById(R.id.item_list_delete_button)
        }
    }

}
```

## Associer l'adapter au recyclerview dans le fragment 

- Modifiez le fragment pour récupérer une instance du recyclerview 

```kotlin
private lateinit var recyclerView: RecyclerView

```

- Modifiez la fonction onCreateView pour instancier le recyclerView 

```kotlin
/**
     * Fonction permettant de définir une vue à attacher à un fragment
     */
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val view = inflater.inflate(R.layout.list_neighbors_fragment, container, false)
        recyclerView = view.findViewById(R.id.neighbors_list)
        recyclerView.layoutManager = LinearLayoutManager(context)
        recyclerView.addItemDecoration(
            DividerItemDecoration(
                requireContext(),
                DividerItemDecoration.VERTICAL
            )
        )
        return view
    }
```

- Surcharges la fonction onViewCreated du fragment pour ajouter l'adapter au recyclerview 

```kotlin 
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    val neighbors = NeighborRepository.getInstance().getNeighbours()
    val adapter = ListNeighborsAdapter(neighbors)
    recyclerView.adapter = adapter
}
```

- Exécutez le projet

> A ce stade, vous devrez avoir une liste avec le nom de tous les voisins mais il manque les photos. 


### Afficher les photos des voisins
> Nous allons modifier l'adapter afin de télécharger la photo de chaque voisin et les afficher dans le layout. 

> Pour cela, nous allons utiliser la librairie Glide 

- Le téléchargement de l'image nécessite un accès à internet. Pour qu'on puisse utiliser internet, il faut ajouter la permission internet dans le manifest. 

```xml
<uses-permission android:name="android.permission.INTERNET" />
```


- Ajouter les dépendances à la librairie Glide dans le gradle du projet 

```gradle 
dependencies {
  ...
  implementation 'com.github.bumptech.glide:glide:4.11.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
  ...
}
```

- Modifier l'adapteur pour télécharger l'image d'un voisin 

```kotlin
val context = [Trouvez le context ]
// Display Neighbour Avatar
Glide.with(context)
    .load(neighbour.avatarUrl)
    .apply(RequestOptions.circleCropTransform())
    .placeholder(R.drawable.ic_baseline_person_outline_24)
    .error(R.drawable.ic_baseline_person_outline_24)
    .skipMemoryCache(false)
    .into(holder.mNeighbourAvatar)

```

- Executez le projet 

> Vous devriez voir les images des utilisateurs qui s'affichent 

### Traiter les événements sur items du recyclerview 
Dans cette section, nous allons traiter les événements sur les élements du recyclerview. Par exemple, quand l'utilisateur clique sur l'icone supprimer on veut afficher une popup demandant à l'utitilisateur de confirmer la suppression. 
Pour y arriver, on doit faire communiquer le fragment ``ListNeighborsFragment`` avec l'adapteur. 

> Pour faciliter la communication entre le fragment et l'adateur, on utilise des une interface qui est implémentée dans le fragment et appelée dans l'adapteur. 

- Ajouter une interface dans le package ``adapters``

```kotlin
    interface ListNeighborHandler {
        fun onDeleteNeibor(neighbor: Neighbor)
    }
```

- Modifiez le fragment ``ListNeighborsFragment`` pour implémenter l'interface ``ListNeighborHandler``

```
Maintenant c'est à vous de jouer
```

- Modifiez le constructeur de l'adapteur pour y ajouter en paramtre une instance de l'interface ``ListNeighborHandler``

```
C'est encore à vous de jouer
```

- Modifiez la fonction ``onBindViewHolder`` dans ``ListNeighborHandler`` pour intercepter le clique sur le bouton ``delete``. 

> Quand on clique sur le bouton, il faut appeler la fonction ``onDeleteNeibor`` de l'interface. 

> Cette action va déclencher l'appel à la fonction ``onDeleteNeibor`` dans le fragment. 

- Modifiez la fonction ``onDeleteNeibor`` dans le fragment pour afficher une dialogue quand l'utilisateur clique sur le bouton supprimer. 

> Infos sur la dialog 
 >> Titre : Confirmation 
 >> Message: Voulez-vous supprimer ce voisin ? 
 >> Bouton 1: Oui
 >> Bouton 2: Non 

> Quand l'utilisateur clique sur oui, vous supprimer le voisin passé en paramtre, sinon vous ne faites rien. 

```
C'est à vous de coder
```


