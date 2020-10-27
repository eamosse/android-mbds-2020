# Application de mise en relation entre voisins

## Partie 2: Ajouter un nouveau voisin
Dans cette section, nous allons ajouter un nouveau fragment qui permettra de créer un nouveau voisin et l'ajouter dans la liste. 

### 1. Créer un nouveau fragment ``AddNeighbourFragment``

1. Ajouter un nouveau fragment dans le projet ainsi que la vue associée

2. Ajouter un layout pour dessiner la vue du fragment ``add_neighbor.xml``

> Vous allez devoir vous débrouller seuls, la vue doit ressembler à l'imge ci-dessous 

![Ajoutez un voisin](/add_neighbour.png "Nouveau voisin")

> Utilisez les composants MaterialComponents, qui permettent de dessiner des composants plus zolies

> Pour plus d'infos sur les composants MaterialComponents  
[voir le lien](https://material.io/develop/android/components)


```xml
<com.google.android.material.textfield.TextInputLayout
    android:id="@+id/addressLyt"
    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="10dp">

    <com.google.android.material.textfield.TextInputEditText
        android:id="@+id/address"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="@string/hint_address"
        android:inputType="text" />
</com.google.android.material.textfield.TextInputLayout>

```

> Dans cet exemple 

>> com.google.android.material.textfield.TextInputLayout permet de définir un layout qui encapsule le champ de texte; il permet entre autre de gérer le floating label. 

>> com.google.android.material.textfield.TextInputEditText
est le champ de texte qui permet à l'utilisateur de saisir le texte. 
 
3. Associer le layout au fragment

4. Gérer l'action sur le bouton enregistre. Quand l'utilisateur clique sur le bouton enregistrer :
    - Récupérer les valeurs de champs 
    - Créer un nouvel objet Neighbor 
    - Ajouter l'objet dans la liste des Neighbors 


### 2. Lancer le fragment ``AddNeighborFragment``
Dans cette section, nous allons modifier l'activité ``MainActivity`` afin de faciliter la gestion du nouveau fragment. 

- Ajouter une interface ``NavigationListener`` dans le package principal du projet. 

```kotlin
    interface NavigationListener {
        fun showFragment(fragment: Fragment)
    }
```

- Modifiez l'activité ``MainActivity`` pour qu'elle implémente l'interface ``NavigationListener``

- Déplacez le code de la méthode ``changeFragment`` dans la nouvelle méthode ``showFragment`` puis supprimer la méthode ``changeFragment``

``` kotlin
class MainActivity : AppCompatActivity(), NavigationListener {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)
        showFragment(ListNeighborsFragment())
    }

    override fun showFragment(fragment: Fragment) {
        supportFragmentManager.beginTransaction().apply {
            replace(R.id.fragment_container, fragment)
            addToBackStack(null)
        }.commit()
    }
}
```
- Modifiez la layout du fragment liste voisins en y ajoutant un bouton floatant (floating button)

```xml
    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="15dp"
        android:layout_marginBottom="15dp"
        android:src="@drawable/ic_baseline_person_add_24"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />
```

- Modifiez le fragment de la liste des voisins pour instercepter le click sur le bouton 

> Au clique sur le bouton, afficher le fragment ``AddNeighborFragment``

```Kotlin
addNeighbor.setOnClickListener { 
            (activity as? NavigationListener)?.let { 
                it.showFragment(AddNeighborFragment())
            }
        }
```

- Compilez et testez 
