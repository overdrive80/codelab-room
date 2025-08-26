summary: Guía de la API Room
author: www.develou.com
id: guia-room
categories: android,room
environments: Android
status: Published
feedback link:

# Guía de la API Room
## Empezando Con Room
Duration: 0:10:00

### Introducción A Room

En este artículo veremos una introducción a Room, una librería de Android que provee una capa de abstracción sobre SQLite, para acceso a bases de datos.

Esta simplifica todas las creaciones del esquema de una base de datos, las relaciones entre ellas y sus operaciones SQL.

Por esta razón aumentará nuestra productividad en comparación al uso del paquete nativo **[android.database.sqlite](https://www.develou.com/android-sqlite-bases-de-datos/)**.

La siguiente es la apertura a una serie de tutoriales para aprender a usar la librería.

### Componentes De Arquitectura

Room es el componente de arquitectura encargado de la persistencia local en la capa de datos de nuestras apps Android.

El siguiente es un diagrama genérico donde vemos como se relaciona con los componentes **[ViewModel](https://www.develou.com/android-viewmodel/)** y **[LiveData](https://www.develou.com/android-livedata/)**.

![MVVC](img/room-componentes-arquitectura-android.png)

Donde el **[Repositorio](https://martinfowler.com/eaaCatalog/repository.html)** es una clase encargada de encapsular la lógica requerida para acceder a las fuentes de datos como **Room**.

Este no hace parte de los componentes de arquitectura de Android, pero es un patrón que Google recomienda usar.

De esta forma los datos fluyen desde los controladores de UI hasta llegar a **SQLite** y operar el conjunto de datos.

Y es justo allí donde estaremos trabajando en esta serie de tutoriales.

###  Componentes De Room

En el diagrama anterior vimos varios objetos asociados a Room. Veamos de qué se tratan.

#### RoomDatabase
Es el punto de entrada principal para comunicar el resto de tu app con el esquema relacional de datos.

Esta clase nos oculta la implementación de **[SQLiteOpenHelper](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper)** para facilitarnos el acceso a la base de datos.

Al extender esta clase veremos algo así:

```java
@Database(entities = {Product.class}, version = 1)
public abstract class ProductsDatabase extends RoomDatabase {
    public abstract ProductDao productDao();
}
```

#### Data Access Object (DAO)

Un DAO representa un mapeado de operaciones SQL a métodos.

Lo que quiere decir que crearemos, leeremos, actualizaremos y eliminaremos registros desde ellos.

Las operaciones se verán así normalmente:

```java
@Dao
public interface ProductDao {
@Query("SELECT * FROM product")
List<Product> getAll();

    @Insert
    void insert(Product product);

    @Delete
    void delete(Product product);
}
```
#### Entity
La clase anotada que describe una tabla de la base de datos en Room.

Es decir, un mapeado del comando CREATE de una tabla en SQLite.

Ejemplo:

```java
@Entity
public class Product {
@PrimaryKey
public String id;

    @ColumnInfo(name = "product_name")
    public String name;

    @ColumnInfo(name = "product_price")
    public float price;
}
```

### Implementar Room

Para usar esta librería seguiremos los siguientes pasos:

1. Añadir dependencia de Room a Gradle
2. Crear una Entity por cada modelo
3. Crear DAOs
4. Crear subclase de RoomDatabase
5. Crear Repositorio
6. Testear capa de datos
7. Testear Migraciones

Veremos cómo aplicar cada uno en esta serie de tutoriales, donde crearemos como **[ejemplo](https://www.develou.com/ejemplo-de-room/)** una App sobre listas de compras.

Adicionalmente cubriremos características como:

- CRUDs
- Relaciones (uno a uno, 1 a muchos, muchos a muchos)
- Relaciones (uno a uno, 1 a muchos, muchos a muchos)
- Búsquedas
- Vistas
- Etc.

<!-- ------------------------ -->
## Crear Una Base De Datos Room
Duration: 0:10:00

En este tutorial vamos a crear una base de datos Room para una App Android de **[ejemplo sobre listas de compras](https://www.develou.com/ejemplo-de-room/)**.

El objetivo es crear los componentes vistos en la **[introducción a Room](https://www.develou.com/introduccion-a-room/)** con el fin de mostrar las listas de compras en un TextView. E ilustrar el funcionamiento de la librería y sus componentes.

![Shoppinglist](img/boceto-shopping-list-app.png)

Puedes descargar el resultado final del tutorial desde el siguiente enlace:

**[Descargar](http://develou.com/downloads/room-1.zip)**

Empecemos el desarrollo.

### 1. Crear Proyecto En Android Studio

1. Abre Android Studio y crea un nuevo proyecto
2. Selecciona una **Empty Activity** y presiona **Next**
3. Nombra al aplicativo como _Shopping List_ y presiona **Finish**.

![Androidstudio](img/androidstudio.png)

### 2. Añadir Dependencias Gradle De Room

1. Abre tu archivo **build.gradle** del módulo y agrega las siguiente dependencias:

    ```java
    dependencies {
    
        implementation "androidx.appcompat:appcompat:$appCompatVersion"
    
        // Room
        implementation "androidx.room:room-runtime:$roomVersion"
        annotationProcessor "androidx.room:room-compiler:$roomVersion"
    
        // Lifecycle
        implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycleVersion"
        implementation "androidx.lifecycle:lifecycle-livedata:$lifecycleVersion"
        implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycleVersion"
    
        // UI
        implementation "com.google.android.material:material:$materialVersion"
        implementation "androidx.constraintlayout:constraintlayout:$contraintLayoutVersion"
    
        // Testing
        testImplementation "junit:junit:$junitVersion"
        androidTestImplementation "androidx.test.ext:junit:$androidxJunitVersion"
        androidTestImplementation "androidx.test.espresso:espresso-core:$espressoVersion"
    }
    ```
2. Ahora abre **build.gradle** del proyecto y crea un bloque llamado `ext` con el número de las versiones:

    ```java
    ext {
    appCompatVersion = '1.2.0'
    
        roomVersion = '2.2.5'
        lifecycleVersion = '2.2.0'
    
        materialVersion = '1.2.1'
        contraintLayoutVersion = '2.0.4'
    
        junitVersion = '4.13.1'
        espressoVersion = '3.3.0'
        androidxJunitVersion = '1.1.2'
    }
    ```   
Puedes obtener el número de versión más reciente desde **[AndroidX releases](https://developer.android.com/jetpack/androidx/versions)**.

### 3. Crear Una Entidad
La primera característica que vamos a desarrollar es ver las listas de compras que existen.

En este momento la entidad tiene una clave primaria y el nombre como se muestra en el siguiente esquema:

![tabla-shopping](img/tabla-shopping-list-bd.png)

**¿Cómo crear una entidad?**

1. Añade la clase `ShoppingList` que represente a las listas de compra. Cada atributo hará referencia a la columna en la tabla.

    ```java
    public class ShoppingList {
    
    
        private final String mId;
        
        private final String mName;
    
        public ShoppingList(@NonNull String id, @NonNull String name) {
            mId = id;
            mName = name;
        }
    
        public String getId() {
            return mId;
        }
    
        public String getName() {
            return mName;
        }
    }
    ```
2. Actualiza el modelo con las siguientes anotaciones:

    ```java
    @Entity(tableName = "shopping_list")
    public class ShoppingList {
    
        @PrimaryKey
        @NonNull
        @ColumnInfo(name = "id")
        private final String mId;
    
        @NonNull
        @ColumnInfo(name = "name")
        private final String mName;
    
        public ShoppingList(@NonNull String id, @NonNull String name) {
            mId = id;
            mName = name;
        }
    
        public String getId() {
            return mId;
        }
    
        public String getName() {
            return mName;
        }
    }
    ```

**¿Cuál es el propósito de cada una?**

- `@Entity`: Marca una clase para ser mapeada por Room como tabla. El nombre por defecto de la tabla será el de la clase. Para especificar uno distinto usa la propiedad `tableName`.
- `@PrimaryKey`: Marca un atributo como la clave primaria
- `@ColumnInfo`: Permite la personalización de la columna asociada con este campo. El atributo name permite cambiar el nombre de la columna (por defecto es el nombre del campo)
- `@NonNull`: Denota que un campo, parámetro o valor de retorno de un método no puede ser `null`. Con ella Room añade la restricción `NOT NULL` a la columna.

### 4. Crear DAO
Un DAO (Data Access Object) u objeto de acceso a datos es la clase principal donde se definen las interacciones con tu base de datos.

Aunque podemos tener uno solo para todos los accesos, es recomendado crear uno por cada tabla operada.

Ahora bien, ¿qué acciones realizaremos en la tabla `shopping_list`?

Por el momento:

- Insertar una lista de compras
- Obtener todas las listas de compras

Vamos a plasmar esto creando una interfaz (también puede ser una clase abstracta) con el nombre de `ShoppingListDao` con el siguiente código:

```java
@Dao
public interface ShoppingListDao {
    @Query("SELECT * FROM shopping_list")
    LiveData<List<ShoppingList>> getAll();

    @Insert(onConflict = OnConflictStrategy.IGNORE)
    void insert(ShoppingList shoppingList);
}
```

Analicemos las partes:

- `@Dao`: Marca la clase como un DAO.
- `@Insert`: Marca un método de un DAO como una operación de inserción. Pasamos como parámetro la entidad (o colección de esta). La propiedad `onConflict` indica que hacer si existe un conflicto al insertar. `IGNORE` ignora el conflicto y mantiene la fila existente.
- `@Query`: Marca un método de un DAO como una consulta. Usamos un `SELECT` para obtener una lista de las entidades consultadas.

Como ves, `getAll()` retorna un **[LiveData](https://www.develou.com/android-livedata/)**. Esto se debe a que Room se las arregla para interactuar con este componente y enviarnos actualizaciones sobre cambios en el esquema de la tabla shopping_list.

En los siguientes tutoriales iremos expandiendo este DAO según lo requiera la incorporación de nuevas funcionalidades.

### 5. Crear Base De Datos Room

Habíamos dicho en la introducción que `RoomDatabase` es la clase base para todas las bases de datos en Room. Así que debes extender tu clase de base de datos de ella.

Esta provee métodos de acceso directo hacia las operaciones de la base de datos Room, sin embargo deberíamos preferir usar los DAOs para ello.

Crea una nueva clase abstracta llamada `ShoppingListDatabase` y agrega el siguiente código:

```java
@Database(entities = {ShoppingList.class}, version = 1, exportSchema = false)
public abstract class ShoppingListDatabase extends RoomDatabase {

    // Exposición de DAOs
    public abstract ShoppingListDao shoppingListDao();

    private static final String DATABASE_NAME = "shopping-list-db";

    private static ShoppingListDatabase INSTANCE;

    private static final int THREADS = 4;

    public static final ExecutorService dbExecutor = Executors.newFixedThreadPool(THREADS);

    public static ShoppingListDatabase getInstance(final Context context) {
        if (INSTANCE == null) {
            synchronized (ShoppingListDatabase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(
                            context.getApplicationContext(), ShoppingListDatabase.class,
                            DATABASE_NAME)
                            .build();
                }
            }
        }
        return INSTANCE;
    }

}
```

Resaltemos las características:

- `@Database`: Marca la clase como una base de datos Room. Usamos las propiedades:
    - `entities`: La lista de entidades incluidas en la base de datos.
    - `version`: Versión de la base de datos.
    - `exportSchema`: Le dice a Room que exporte el esquema.
- **Exposición de DAOs**: Creamos un método `get*()` abstracto por cada DAO que tengamos.
- **Singleton**: Usamos este **[patrón](https://es.wikipedia.org/wiki/Singleton)** si queremos una sola instancia de la base de datos abierta (`INSTANCE` y `getInstance()`)
- **Hilos**: Declaramos un `ExecutorService` para ejecutar las operaciones de bases de datos en otros hilos de trabajo y por ende no entorpecer la UI.
- **Creación de la base de datos**: Pasamos al método `Room.databaseBuilder()` el contexto de la aplicación, el tipo de la clase de base de datos y el nombre de la base de datos. Al final usamos `build()` y nuestra instancia se creará.

### 6. Crear Repositorio

El siguiente paso es crear la clase del repositorio de las listas de compras.

**¿Qué debemos tener en cuenta?**

- Declaramos la dependencia al DAO relacionado con las listas de compra
- Declaramos un campo para el **[LiveData](https://developer.android.com/topic/libraries/architecture/livedata)** de las listas de compra
- Añadimos un método por cada operación que nos interese realizar en la tabla (`insert()` y `getAllShoppingLists()`)

Crea la clase `ShoppingListRepository` y pega el siguiente código:

```java
public class ShoppingListRepository {
private final LiveData<List<ShoppingList>> mShoppingLists;
private final ShoppingListDao mShoppingListDao;

    public ShoppingListRepository(Context context) {
        ShoppingListDatabase db = ShoppingListDatabase.getInstance(context);
        mShoppingListDao = db.shoppingListDao();
        mShoppingLists = mShoppingListDao.getAll();
    }

    public LiveData<List<ShoppingList>> getAllShoppingLists() {
        return mShoppingLists;
    }

    public void insert(ShoppingList shoppingList) {
        ShoppingListDatabase.dbExecutor.execute(
                () -> mShoppingListDao.insert(shoppingList)
        );
    }
}
```

En el constructor ordenaremos una carga automática de las listas de compra con el fin de recibir las notificaciones de datos lo más pronto posible.

De esta forma, Room envía a otro hilo de trabajo a las consultas con `LiveData` y se asegura de comunicar los cambios a la UI.

En el caso de la inserción debemos llamar al `ExecutorService` para ejecutarla en su hilo correspondiente.

### 7. Crear ViewModel

Para crear el **[ViewModel](https://www.develou.com/android-viewmodel/)** nospreguntamos:

**¿Cuáles son las interacciones que vendrán desde la interfaz?**

En este momento solo vamos a insertar un par de registros programaticamente y los mostraremos en el `TextView` que aparece en el layout creado automáticamente por la plantilla **Empty Activity**.

Por lo que observaremos la lista de listas de compras con un `LiveData` desde `MainActivity`.

Esto significa que envolveremos a los métodos `getAllShoppingLists()` e `insert()` del repositorio en los del `ViewModel`.

Teniendo en mente lo anterior, crea la clase `ShoppingListViewModel` y materializa estas características:

```java
public class ShoppingListViewModel extends AndroidViewModel {

    private final ShoppingListRepository mRepository;

    private final LiveData<List<ShoppingList>> mShoppingLists;

    public ShoppingListViewModel(@NonNull Application application) {
        super(application);
        mRepository = new ShoppingListRepository(application);
        mShoppingLists = mRepository.getAllShoppingLists();
    }

    public LiveData<List<ShoppingList>> getShoppingLists() {
        return mShoppingLists;
    }

    public void insert(ShoppingList shoppingList) {
        mRepository.insert(shoppingList);
    }
}
```

En el constructor cargaremos todas las listas de compra para actualizar la vista inmediatamente.

### 8. Prepoblar La Base De Datos

Si queremos que la base de datos comience con registros predefinidos podemos hacer uso de `RoomDatabase#Callback`.

Crea una `callback` en la clase de base de datos y sobrescribe su método `onCreate()` para agrega 2 listas de compras:

```java
// Prepoblar base de datos con callback
private static final RoomDatabase.Callback mRoomCallback = new Callback() {
    @Override
    public void onCreate(@NonNull SupportSQLiteDatabase db) {
        super.onCreate(db);
    
        dbExecutor.execute(() -> {
            ShoppingListDao dao = INSTANCE.shoppingListDao();

            ShoppingList list1 = new ShoppingList("1", "Lista de ejemplo");
            ShoppingList list2 = new ShoppingList("2", "Banquete de Navidad");

            dao.insert(list1);
            dao.insert(list2);
        });
     }
};
```    

Luego la añádela con `addCallback()` en el `builder`.

```java 
INSTANCE = Room.databaseBuilder(
                context.getApplicationContext(), ShoppingListDatabase.class,
                DATABASE_NAME)
                .addCallback(mRoomCallback)
                .build();
```

### 9. Mostrar Datos En La Actividad
La actividad creada por defecto trae consigo un layout con un `TextView`. Algo así:

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:layout_height="match_parent"
tools:context=".MainActivity">

    <TextView
        android:id="@+id/db_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

Finalizando, conecta ese `view` con los datos de la tabla.

Es decir, obtenemos el `ViewModel` en el método `onCreate()` y observamos su contenido con `observe()`:

```java
public class MainActivity extends AppCompatActivity {

    private ShoppingListViewModel mViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView dbText = findViewById(R.id.db_text);

        ViewModelProvider.AndroidViewModelFactory factory =
                ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication());

        mViewModel = new ViewModelProvider(this, factory)
                .get(ShoppingListViewModel.class);

        mViewModel.getShoppingLists().observe(this, shoppingLists -> {
                    StringBuilder sb = new StringBuilder();
                    for (ShoppingList list : shoppingLists) {
                        sb.append(list.getName()).append("\n");
                    }
                    dbText.setText(sb.toString());
                }
        );
    }
}
```

Al usar un `StringBuilder` podemos concatenar el nombre de cada lista y proyectarlo en el texto con `setText()`.

Y para terminar, ejecuta el proyecto. Deberías ver lo siguiente:

<img src="img/screenshot-app-shopping-list-616x1300.png" width="300" alt="captura_lista">