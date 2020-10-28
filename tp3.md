# Application de mise en relation entre voisins

## Partie 2: Ajouter un nouveau voisin
Dans cette section, nous allons ajouter un nouveau fragment qui permettra de créer un nouveau voisin et l'ajouter dans la liste. 

### 1. Créer un nouveau fragment 

1. Ajouter un nouveau fragment ``AddNeighbourFragment`` dans le projet ainsi que la vue associée

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

> Contraintes 

>> Valider les données des champs du formulaire 

    - Tous les champs sont obligatoires

    - L'email doit être valide (afficher une erreur sous le champ email indiquant à l'utilisateur qu'il y a une erreur)

    - Le bouton reste grisé tant que tous les champs ne sont pas validés
    
    - Le téléphone doit être valide (commencant par 06 ou 07 et doit avoir 10 charactères) -- (afficher une erreur sous le champ téléphone indiquant à l'utilisateur que le format doit être 0X XX XX XX XX XX)

    - Le champ bio A propos de moi doit autoriser au maximum 30 charactères 

    - Les champs Image et Website doivent être des liens valides

>> Bonus, quand l'utilisateur rempli le champ Image avec un lien valide, afficher automatiquement un visuel de l'image à la place de l'image par défaut tout en haut. 

<details>
<summary>Besoin d'aide pour la gestion d'erreurs ou le nombre de charactères max ?</summary>
C'est par ici, https://material.io/develop/android/components/text-fields.
</details>

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

## Partie 3 : Gestion de la toolbar 
Dans cette section, nous allons gérer le thème et styles de l'application ainsi que la toolbar de navigation. 

En effet, jusqu'ici notre application n'affiche pas le titre des écrans ou les options de navigation. Nous allons réctifier cela en ajoutant une toolbar. 

### Ajouter une toolbar 
Dans une application Android, une toolbar permet gérer la barre d'action modélisation le titre des écrans, ou les options de navigations. 

- Modifiez le layout de l'activité principal pour y ajouter une toolbar. 
> La toolbar doit se positionner tout en haut de l'écran et les autres vues doivent se positionner sous la toolbar. 

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:layout_weight="1"
        android:background="?attr/colorPrimary"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_scrollFlags="scroll|enterAlways"
        app:title="@string/app_name" />

    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/toolbar_activity_add" />

</androidx.constraintlayout.widget.ConstraintLayout>

```

- Modifiez l'activité pour qu'elle utilise la toolbar pour gérer la navigation 
```kotlin
...
private lateinit var toolbar: Toolbar
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    setContentView(R.layout.activity_main)
    toolbar = findViewById(R.id.toolbar)
    setSupportActionBar(toolbar)
    
    showFragment(ListNeighborsFragment())
}
...
```

- Ajoutez une fonction setTitle dans l'interface ``NavigationListener`` permettant aux fragments de modifier le titre de la toolbar

```kotlin
interface NavigationListener {
    fun showFragment(fragment: Fragment)
    fun updateTitle(@StringRes title: Int)
}
```

- Modifiez l'activité pour changer le titre de la toolbar à l'appel de la fonction ``updateTitle``

```kotlin
...
override fun updateTitle(title: Int) {
    toolbar.setTitle(title)
}
...

```

- Modifiez les fragments ``ListNeighborsFragment`` et ``AddNeighborFragment`` pour afficher le bon titre dans la toolbar : 
    - ListNeighborsFragment --> Liste des voisins
    - AddNeighborFragment --> Nouveau voisin

- Exécutez et vérifier que tout va bien 

- Compilez et testez 

## Partie 4 : Internationalisation de l'application
On va traduire notre application, initialement en français, en anglais. 

- Dans le pannel de gauche, passer sur le mode Android (si ce n'est pas déjà le cas)

- Faites clique droit sur le dossier values puis ``new > Values Resource File ``

- Dans la fenêtre qui s'affiche, ``entrez strings`` dans le champ file Name

- Dans la section ``Available qualifiers``, cliquez Locale puis cliquez sur le bouton ``>>`` pour le selectionner

- Sous Language, selectionnez ``en`` puis cliquez sur ``Ok``

> Une fois remplie, le formulaire devrait ressembler à 

![Add language](/language.png "Add language")

> Si vous déployez le dossier strings dans values, vous devriez voir deux fichiers ``strings.xml`` dont l'un avec ``en entre parenthèse``

>> strings.xml contient les textes par défaut de l'application (en français)

>> strings.xml (en) contient les textes en anglais (il est vide pour l'instant)

- Copiez tout le contenu du fichier strings.xml et collez le dans le fichier strings.xml (en)

- Traduisez les valeurs de chaque item en anglais 

- Allez dans les settings du téléphone (ou de l'émulateur) puis changez la langue du français à l'anglais, ou inversement

- Exécutez le projet, normalement les textes sont traduits en fonction de la langue du téléphone

> Si vous utilisez une langue non supportée (ex. Espagnol), les textes de l'application seront en français

> Vous pouvez ajouter autant de langues que vous souhaitez en répétant la même démarche. 
