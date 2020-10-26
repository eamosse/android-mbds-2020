# Les activités
> Dans une application Android, une activité est le composant qui gère l'interface présentée à l'utilisateur.

> La création ainsi que la gestion du cycle de vie d'une activité sont traitées intégralement par le système d'exploitation Android. Ainsi, le developpeur ne peut pas créer implicitement une activité, ni la détruire; c'est du ressort du systme d'exploitation. Les activités sont créées via des intentions. 

## 1. Création d'un nouveau module 
Dans le projet, créé durant le cours, on va ajouter un nouveau module de type application. 

- Avant tout, commencons par changer le mode d'affichage des sources du projet; on va passer du mode Android au mode Project. 
Dans le panel de gauche, cliquez sur l'icone de la mascotte Android et choisissez Project. 
- Faites bouton droit sur le nom du projet puis cliquez sur **new > Module**
- Dans l'écran qui s'affiche, choisissez **Phone & Tablet Module**
- Dans le champ nom du module, tapez **activities**
    > Assurez-vous d'avoir selectionné Kotlin dans l'option language
- Cliquez sur le bouton suivant 
- Plusieurs templates d'application vous sont présentés, choisissez **Empty Activity**
- Appuez sur **Next**
- Laissez le nom de l'activité (MainActivity) ainsi que le layout (main_activity)
- Cliquez sur **Finish**
  > A ce stated, vous devez avoir un autre module d'application dans votre projet et il est grand temps de vérifier que tout va bien. 
  
- Selectionnez le module `activities`, choisissez le téléphone sur lequel vous voulez déployer le projet, puis cliquez sur run (le bouton play vert) 

![Compilez](/run.png "Compilez le projet")

## 2. Modifiez la vue (activity_main)
- Allez dans le dossier **res > layout** puis cliquez sur le fichier **activity_main**
- Remplacez le text (Hello world!) par un bouton (Cliquez-moi)
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/btn_click_me"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Cliquez moi"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```
> Notez la propriété id du bouton; celle-ci est nécessaire car il permettra de récupérer le bouton dans le code l'activité

> Compilez et assurez-vous qu'un bouton s'affiche au milieu de la vue

## 3. Traitez l'action clique sur le bouton 
On va modifier le code de l'activité afin de récupérer l'instance du bouton pour y ajouter le code à exécuter à l'action du clique par l'utilisateur. 

- Ajoutez un attribut de type **Button** dans la classe **MainActivity**
```Kotlin
    private lateinit var clickButton: Button
```
> Remarquez l'utilisation du lateInit sur le bouton. On ne dispose pas de tous les éléments nécessaires pour la création du bouton, par conséquent nous ne pourrons pas l'initialiser lors de la création de l'activité. Le mot clef lateInit permet d'indiquer au compilateur, qu'on le fera plus tard, c'est-à-dire quand la vue est créée. 

- Initialisez le bouton. Dans la méthode onCreate (juste après le setContentView), recupérez initialisez la variable clickButton. 
```Kotlin
    clickButton = findViewById(R.id.btn_click_me)
```
> Ici on a utilisé la fonction findViewById, qui permet de récupérer l'instance d'une vue à partir de son id. 

> Remarquez aussi comment on accède à l'id d'un objet R.id.btn_click_me

- Interceptez le clique sur le bouton et affichez un toast 
> En Android, un toast est un text qui s'affiche tout en bas de l'écran puis disparait automatiquement qq secondes après (en fonction de la durée définie).

```kotlin 
clickButton.setOnClickListener {
    Toast.makeText(baseContext, "Tu m'as cliqué", Toast.LENGTH_LONG).show()
}
```

- Exécutez le projet, puis cliquez sur le bouton.

> Que remarquez-vous ? 

## 4. Jouons un peu avec le code 
- Commentez la ligne qui initialise le bouton 
```kotlin
    //clickButton = findViewById(R.id.btn_click_me)
```
- Exécutez le projet 
> Pourquoi il n'y a pas d'erreur lors de la compilation ? 

- Cliquez sur le bouton 
> Qu'avez-vous observez ? 

> Pourquoi l'application a planté ? 

> Avez-vous compris le danger des lateInit ? 

## 5. Comptez le nombre de fois qu'on a cliquez sur le bouton 
Dans cette section, nous allons modifier l'action du clique sur le bouton pour afficher le nombre de fois que l'utilisateur a cliquer sur le bouton (au lieu d'un toast).

- Créez une variable qui représente l'occurence du nombre de clique par l'utilisateur 
```kotlin
    private var nbClick = 0 
```

- Modifiez les instructions dans le listener du click sur le bouton pour modifier le texte du bouton en y affichant le nombre de clique en plus du text (Cliquez-moi) 

```kotlin
    clickButton.setOnClickListener {
        nbClick++
        val newText = "Cliquez moi $nbClick"
        clickButton.text = newText
    }
```

- Exécutez le projet puis cliquez sur le bouton

## 6. Exercice : Ajoutez un Textview pour afficher le nombre de fois qu'on a cliqué sur le bouton 

Dans cette section, vous allez modifier le layout de l'activité pour y ajouter un label (TextView) qui affichera le nombre de fois que l'utilisateur a cliqué sur le bouton. 

- Ajoutez le TextView dans le layout 
- Modifiez le code de l'activité pour récupérer l'instance du TextView 
- Modifiez l'action du bouton pour afficher le texte (Vous avez cliquez x fois) 
- Contraintes : 
    - Ne pas afficher le TextView si le nombre de clique est 0
    - Désactivez le bouton (le griser) après 5 cliques




