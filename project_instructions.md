# Mise en place de l'architecture du mini projet 
Afin de s'assurer que tout le monde parte sur les mêmes bases et surtout qu'on respecte au mieux les bonnées pratiques, nous allons construire ensemble l'architecture du mini projet MBDS News. 

## Créer un nouveau projet 
- Dans Android Studio, créez un nouveau projet ``Newsletter``
- Assurez-vous que le nom de package est unique
- Selectionnez Empty Activity comme template 
- Choisir Kotlin comme langage
- Activer la gestion de version
- Commiter les premiers fichers
- Partager le lien avec les copains du groupe

## Mise en place du service 
Dans cette section, nous allons mettre en place le service permettant de récupérer les données depuis l'API https://newsapi.org/


- Ajoutez la permission internet dans le manifest
- Ajoutez les dépendances aux librairies Retrofit, OkHTTP et Gson
```gradle
//Retrofit : Pour consommer les web services RestFul
implementation "com.squareup.retrofit2:retrofit:2.6.0"
//okhttp : Effectuer les requêtes HTTP (utilisée par Retrofit)
implementation "com.squareup.okhttp3:okhttp:3.12.0"
//Gson: Convertir les Json en objet POJ
implementation "com.google.code.gson:gson:2.8.5"
//Gson Converter : Utilisé par Retrofit pour convertir les objets JSON en POJO
implementation "com.squareup.retrofit2:converter-gson:2.4.0"
//Retrofit Logger: Permet d'arricher les logs des requêtes
implementation "com.squareup.okhttp3:logging-interceptor:4.2.1"
//Coroutines : Pour faire du multithreading
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.7"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.7"
implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
```
- Créez un sous-package data
> On va placer dans ce package toutes les classes permettant de traiter les données de l'application. 
- Ajoutez une interface ApiService listant toutes les fonctionnalités de l'application necessitant des données. 
> Pour l'instant, on va ajouter uniquement une méthode getArticles(), vous pourrez le modifier après en fonction des besoins. 

> Créer une classe ArticleResponse modélisant un objet article telque retourné par le webservice
```kotlin
data class ArticleResponse(
val status: String, 
val totalResults: Int, 
val articles: List<Article>
)

data class Article(
val author: String, 
val title: String, 
val description: String,
/**Ajouter les autres attributs **/
)
```

```kotlin
interface ArticleService {
    fun getArticles(): List<Article>
}
```

- Configurer Retrofit 
> Retrofit utilise aussi des interfaces pour modéliser les différentes actions d'une API REST. 
- Ajoutez une interface ``RetrofitApiService``
```kotlin
interface RetrofitApiService {
    //GET --> On lance une requête de type GET
    // everything est l'action du web service qu'on veut apeler
    // Elle sera concaténée avec l'url prédéfini dans retrofit 
    @GET("/everything")
    fun list(): Call<List<ArticleResponse>>
}
```

- Ajoutez une implémentation de l'interface ``ArticleService``, que vous pouvez appeler ``ArticleOnlineService`` pour récupérer les articles depuis le web serivice 

```kotlin

class ArticleOnlineService : ArticleService {
    private val service: RetrofitApiService

    init {
        val retrofit = buildClient()
        //Init the api service with retrofit
        service = retrofit.create(RetrofitApiService::class.java)
    }

    /**
     * Configure retrofit
     */
    private fun buildClient(): Retrofit {
        val httpClient = OkHttpClient.Builder().apply {
            addLogInterceptor(this)
            addApiInterceptor(this)
        }.build()
        return Retrofit
            .Builder()
            .baseUrl(apiUrl)
            .addConverterFactory(GsonConverterFactory.create())
            .client(httpClient)
            .build()
    }

    /**
     * Add a logger to the client so that we log every request
     */
    private fun addLogInterceptor(builder: OkHttpClient.Builder) {
        val httpLoggingInterceptor = HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        }
        builder.addNetworkInterceptor(httpLoggingInterceptor)
    }

    /**
     * Intercept every request to the API and automatically add
     * the api key as query parameter
     */
    private fun addApiInterceptor(builder: OkHttpClient.Builder) {
        builder.addInterceptor(object : Interceptor {
            override fun intercept(chain: Interceptor.Chain): Response {
                val original = chain.request()
                val originalHttpUrl = original.url
                val url = originalHttpUrl.newBuilder()
                    .addQueryParameter("apikey", apiKey)
                    .build()

                val requestBuilder = original.newBuilder()
                    .url(url)
                val request = requestBuilder.build()
                return chain.proceed(request)
            }
        })
    }

    override fun getArticles(): List<Article> {
        return service.list().execute().body()?.articles ?: listOf()
    }

    companion object {
        private const val apiKey = "YOUR_API_KEY"
        private const val apiUrl = "THE_API_URL"
    }

}
```

## Consommer le service dans un fragment 
L'accès à internet bloque le thread principale, c'est en effet une opération entrée/sortie via le protocole http. Par conséquent, les appels aux différentes méthodes du service doivent se faire dans un thread seconde. 

En kotlin, les coroutines peuvent être utilisés pour gérer plus aisément les threads secondaires et les tâches asynchrones. 

Pour avoir plus de détails sur les coroutines, consultez ce lien https://kotlinlang.org/docs/reference/coroutines/coroutines-guide.html 

Dans un fragment, il suffit d'utiliser l'objet ``lifecycleScoep`` pour changer de thread, passé du thread principale à un thread secondaire et inversément. 

- Récupérer la liste des articles dans un fragment 

```kotlin
/**
     * Récupère la liste des articles dans un thread secondaire 
     */
    private fun getArticles() {
        lifecycleScope.launch(Dispatchers.IO) {
            val articles = ArticleRepository.getInstance().getArticles()
        }
    }

    /**
     * Rempli le recyclerview avec les données récupérées dans le web service 
     * Cette action doit s'effectuer sur le thread principale 
     * Car on ne peut mas modifier les éléments de vue dans un thread secondaire 
     */
    private fun bindData(articles: List<Article>) {
        lifecycleScope.launch(Dispatchers.Main) {
            //créer l'adapter 
            //associer l'adapteur au recyclerview 
        }
    }

```

## Sauvegarder des données dans une base de données locale 
https://developer.android.com/topic/libraries/architecture/room?gclid=CjwKCAjw0On8BRAgEiwAincsHBXJ_ek_uEx0SCQsdUeh6IVIUuSmtrkHynCnqc9FgyUlnt7la6n_mBoCcIQQAvD_BwE&gclsrc=aw.ds 
