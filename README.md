# Les activités
> Dans une application Android, une activité est le composant qui gère l'interface présentée à l'utilisateur.

La création ainsi que la gestion du cycle de vie d'une activité sont traitées intégralement par le système d'exploitation Android. Ainsi, le developpeur ne peut > pas créer implicitement une activité, ni la détruire; c'est du ressort du systme d'exploitation. Les activités sont créées via des intentions. 

1. Création d'un nouveau module 
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
  Pour cela, repérez le petit marteau (vert) et selectionnez dans le champ à sa droite le module que vous venez de créer puis cliquez sur le signe play 

2. Compilez le projet 
