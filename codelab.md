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
## Crear Una Base De Datos Room <a name="crear-bd"></a>
Duration: 0:10:00

En este tutorial vamos a crear una base de datos Room para una App Android de **[ejemplo sobre listas de compras](https://www.develou.com/ejemplo-de-room/)**.

El objetivo es crear los componentes vistos en la **[introducción a Room](https://www.develou.com/introduccion-a-room/)** con el fin de mostrar las listas de compras en un TextView. E ilustrar el funcionamiento de la librería y sus componentes.

![Shoppinglist](img/boceto-shopping-list-app.png)

Puedes descargar el resultado final del tutorial desde el siguiente enlace:

<button>[Descargar](http://develou.com/downloads/room-1.zip)</button>

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

## Insertar Datos Con Room
En este tutorial verás cómo insertar datos con Room desde una actividad de creación de listas de compras.

Recuerda que puedes ver el alcance general de este ejemplo en la [**descripción de la App**](https://www.develou.com/ejemplo-de-room/).

Al haber [**creado la base de datos**]() de listas de compra, ahora agregaremos un `RecyclerView` para mejorar la vista de `MainActivity`.

Adicionalmente, pondremos un `FAB` que inicie una nueva actividad de creación. El siguiente boceto muestra nuestro plan:

![insertar_lista](img/insertar-lista-de-compras-app.png)

Puedes descargar el código completo desde el siguiente link:

<button>[Descargar](http://develou.com/downloads/room-2-nKtrws8xNUej5EdLl70nZw.zip)</button>

Codifiquemos la solución.

### 1. Añadir RecyclerView Al Layout
Reemplazar el layout actual requiere las siguientes acciones:

- Reemplazar `ConstraintLayout` raíz por `CoordinatorLayout`.
- Reemplazar `TextView` por `RecyclerView`.
- Añadir `FAB` en la parte inferior derecha.
- Añadir icono con símbolo ‘+’.
- Añadir soporte de vectores.

Basado en ello, abre el layout `activity_main.xml` y pega el siguiente código:

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:layout_height="match_parent"
tools:context=".shoppinglists.MainActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/floating_action_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="16dp"
        app:srcCompat="@drawable/ic_add_24"/>
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

Para agregar el icono haz clic derecho en tu carpeta **drawable**, presiona **New**, luego **Vector Asset**. Seguido selecciona **Clip Art** en **Asset Type**, escribe **«add»** en el diálogo emergente y selecciona el icono que necesitamos.

![vector_asset](img/vector_asset.png)

Nómbralo **ic_add_24** y confirma su inclusión.

Ahora abre el archivo **build.gradle** del módulo y pega la siguiente instrucción para habilitar el uso de vectores en el proyecto Android Studio:

```
defaultConfig {
    vectorDrawables.useSupportLibrary = true
}
```
Sincroniza y tendremos completa nuestra interfaz para mostrar las listas de compras.

### 2. Crear Adaptador Del RecyclerView
El **layout** para el ítem de la lista es simple. Crea el archivo `shopping_list_item.xml` y recubre un `TextView` con un `ConstraintLayout`:

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:padding="@dimen/normal_padding"
android:layout_height="?listPreferredItemHeight">

    <TextView
        android:id="@+id/name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="Lista de ejemplo" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

A continuación, crea la clase `ShoppingListAdapter`.

La idea es _inflar_ el layout del ítem. Agregar un método para actualizar los ítems y bindear el nombre de las listas de compra con el `TextView` existente.

**Solución:**

```java
public class ShoppingListAdapter
extends RecyclerView.Adapter<ShoppingListViewHolder> {

    private List<ShoppingList> mShoppingLists;

    @NonNull
    @Override
    public ShoppingListViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        return ShoppingListViewHolder.create(parent);
    }

    @Override
    public void onBindViewHolder(@NonNull ShoppingListViewHolder holder, int position) {
        ShoppingList item = mShoppingLists.get(position);
        holder.bind(item);
    }

    @Override
    public int getItemCount() {
        return mShoppingLists == null ? 0 : mShoppingLists.size();
    }

    public void setItems(List<ShoppingList> items) {
        mShoppingLists = items;
        notifyDataSetChanged();
    }

}
```

Crea también la clase `ShoppingListViewHolder` para procesar el _inflado_ y el _binding_ de cada ítem:

```java
public class ShoppingListViewHolder extends RecyclerView.ViewHolder {
private final TextView mNameText;

    public ShoppingListViewHolder(@NonNull View itemView) {
        super(itemView);
        mNameText = itemView.findViewById(R.id.name);
    }

    public void bind(ShoppingList item) {
        mNameText.setText(item.getName());
    }

    public static ShoppingListViewHolder create(ViewGroup parent) {
        View v = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.shopping_list_item, parent, false);
        return new ShoppingListViewHolder(v);
    }
}
```

### 3. Poblar RecyclerView Desde Room
**¿Cómo ponemos el contenido de la base de datos en la lista?**

- Abre `MainActivity`.
- Obtén la instancia del `RecyclerView`.
- Crea una instancia del **Adaptador** y asígnala al **recycler**.
- Dirígete donde observamos el `LiveData` del `ViewModel`. Reemplaza ese código por la llamada del método `setItems()` del adaptador.

Lo anterior reflejado en código sería así:

```java
public class MainActivity extends AppCompatActivity {

    private ShoppingListViewModel mViewModel;
    private RecyclerView mList;
    private ShoppingListAdapter mAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ViewModelProvider.AndroidViewModelFactory factory =
                ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication());

        mViewModel = new ViewModelProvider(this, factory)
                .get(ShoppingListViewModel.class);

        setupList();

        setupFab();
    }

    private void setupList() {
        mList = findViewById(R.id.list);
        mAdapter = new ShoppingListAdapter();
        mList.setAdapter(mAdapter);
        mViewModel.getShoppingLists().observe(this, mAdapter::setItems);
    }

    private void setupFab() {
        findViewById(R.id.floating_action_button)
                .setOnClickListener(view -> addNewShoppingList());
    }

    private void addNewShoppingList() {
        startActivity(new Intent(this, AddShoppingListActivity.class));
    }
}
```

Si ejecutas el proyecto en este punto deberías ver este resultado:

<img src="img/room-recyclerview-616x1300.png" width="300" alt="recycler_room">

### 4. Añadir Actividad De Creación De Listas De Compras
Para la actividad de **«Nueva lista»** usaremos una plantilla **Empty Activity**. El nombre de la clase será `AddShoppingListActivity`.

El **layout** consta de un `EditText` y un botón de **guardar**. Abre `activity_add_shopping_list.xml` para satisfacer el diseño:

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:padding="@dimen/normal_padding"
tools:context=".addshoppinglist.AddShoppingListActivity">

    <Button
        android:id="@+id/create_button"
        style="?attr/materialButtonOutlinedStyle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:text="@string/create_button"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="1.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/name_layout" />

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/name_layout"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/shopping_list_hint"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <com.google.android.material.textfield.TextInputEditText
            android:layout_width="match_parent"
            android:id="@+id/name_field"
            android:layout_height="wrap_content" />
    </com.google.android.material.textfield.TextInputLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

También debemos asegurarnos de retornar con el **Up Button**. Por lo que habilitamos su aparición con en `onCreate()`:`

```java
public class AddShoppingListActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_shopping_list);

        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
    }

    

    @Override
    public boolean onSupportNavigateUp() {
        onBackPressed();
        return true;
    }
}
```

### 5. Guardar Lista Nueva En Base De Datos
Al presionar el botón de **crear**, debemos comunicárselo al `ViewModel` que es el encargado de insertar datos con Room con su método `insert()`.

**¿Cómo lo haces?**

- Obtén la instancia de `ShoppingListViewModel` en `onCreate()`.
- Obtén la instancia del boton de **crear**.
- Asigna un objeto `OnClickListener` al botón.
- Llama a `insert()` del view model en `onClick()`.
- Finaliza la actividad.

Al completar las tareas anteriores tu actividad de creación se verá así:

```java
public class AddShoppingListActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_shopping_list);

        getSupportActionBar().setDisplayHomeAsUpEnabled(true);

        ViewModelProvider.AndroidViewModelFactory factory
                = ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication());
        ShoppingListViewModel vm = new ViewModelProvider(this, factory)
                .get(ShoppingListViewModel.class);

        setupCreateButton(vm);
    }

    private void setupCreateButton(ShoppingListViewModel vm) {
        findViewById(R.id.create_button).setOnClickListener(
                view -> {
                    // Obtener valor del campo de texto
                    EditText nameField =  findViewById(R.id.name_field);
                    String name = nameField.getText().toString();

                    // Ignorar acción si hay 0 caracteres
                    if (name.isEmpty()) {
                        return;
                    }

                    // Crear entidad y guardarla
                    String id = UUID.randomUUID().toString();
                    ShoppingList shoppingList = new ShoppingList(id, name);
                    vm.insert(shoppingList);

                    // Ir a la lista
                    finish();
                });
    }

    @Override
    public boolean onSupportNavigateUp() {
        onBackPressed();
        return true;
    }
}
```

No olvides iniciar esta actividad al presionar el `FAB` de la lista:

```java
private void setupFab() {
    findViewById(R.id.floating_action_button)
            .setOnClickListener(view -> addNewShoppingList());
}

private void addNewShoppingList() {
    startActivity(new Intent(this, AddShoppingListActivity.class));
}
```

Ejecuta el proyecto y comprueba la inserción de listas.

<img src="img/screenshot-crear-lista-compras-app-616x1300.png" width="300" alt="crear_listas"><img src="img/screnshot-lista-creada-app-616x1300.png" width="300" alt="lista_creada">

> aside positive **Nota**: Aunque en este ejemplo estamos usando un solo `ViewModel`, normalmente debes crear uno por cada controlador de vista.

### 6. Insertar Una Lista De Entities
Room tiene la capacidad de [**insertar múltiples entidades**](https://developer.android.com/training/data-storage/room/accessing-data#convenience-insert) si pasamos una lista como parámetro.

Para ver su funcionamiento agreguemos a `ShoppingListDao` el siguiente método:

```java
@Insert(onConflict = OnConflictStrategy.IGNORE)
void insertShoppingLists(List<ShoppingList> lists);
```

Ahora desde `ShoppingListDatabase` creamos una lista de 5 elementos para reemplazar la prepoblación que hacíamos antes de forma individual:
        
```java
// Prepoblar base de datos con callback
private static final RoomDatabase.Callback mRoomCallback = new Callback() {
    @Override
    public void onCreate(@NonNull SupportSQLiteDatabase db) {
        super.onCreate(db);

        dbExecutor.execute(() -> {
            ShoppingListDao dao = INSTANCE.shoppingListDao();

            List<ShoppingList> lists = new ArrayList<>();
            for (int i = 0; i < 5; i++) {
                String id = UUID.randomUUID().toString();
                lists.add(new ShoppingList(id, "Lista " + (i+1)));
            }

            dao.insertShoppingLists(lists);
        });
    }
};
```

De esta forma, podremos hacer en una misma transacción un **bulk** de inserciones de listas de compras.

## Consultar Datos Con Room
En este tutorial veremos con más detalle como consultar datos con Room y la anotación `@Query`.

> aside positive **Más específicamente**: consultar pasando uno o múltiples parámetros y como consultar solo las columnas que necesitemos.

¿Cómo lo reflejaremos en nuestra [**App de listas de compras**]()?

Añadiremos un filtro simple en la actividad principal y crearemos una actividad de edición como lo ilustra el siguiente boceto:

![consultar_datos_room](img/consultar-datos-con-room.png)

Puedes descargar el código completo del proyecto desde el siguiente enlace:

<button>[Descargar](http://develou.com/downloads/room-3-3ebCorpdZkStjqMyKYO1og.zip)</button>

Veamos de qué va la solución.

### 1. Enlazar Un Parámetro En Una Consulta
Los métodos anotados con `@Query` en nuestros DAOs pueden recibir parámetros con el fin de enlazarlos con argumentos de la consulta.

Para ello usamos la sintaxis `:nombre` con el fin de diferenciar el contexto.

**Ejemplo:**

Podemos añadir como parámetro el **ID** que proporcione el item cliqueado en el **recyclerView** para consultar la lista de compras.

Abre `ShoppingListDao` y crea un nuevo método llamado `getShoppingList()` que reciba el **ID** y _bindealo_ así:

```java
@Query("SELECT * FROM shopping_list WHERE id = :id LIMIT 1")
LiveData<ShoppingList> getShoppingList(String id);
```

De esta forma, `:id` será reemplazado por el parámetro `id`.

### 2. Enlazar Múltiples Parámetros En Una Consulta
Con la misma lógica, Room también soporta el enlace de una lista de parámetros. La librería genera automáticamente la consulta en tiempo de ejecución, asignando cada elemento en la sentencia.

**Ejemplo:**

Probemos esta característica con los filtros por categoría propuestos en el boceto al inicio del tutorial.

Lo primero será agregar la categoría como columna a la entidad `ShoppingList` así:

```java
@Nullable
@ColumnInfo(name = "category")
private final String mCategory;
```

Ahora crea un nuevo método en el DAO que se llame `getShoppingListsByCategories()` y pásale una lista de strings como parámetro. Bindeala en un operador **IN**:

```java
@Query("SELECT * FROM shopping_list WHERE category IN(:categories)")
LiveData<List<ShoppingList>> getShoppingListsByCategories(List<String> categories);
```

Y listo, de esa forma consultaremos los elementos de la actividad principal cuando agreguemos los check boxes para filtrar.

### 3. Seleccionar Un Subconjunto De Columnas En Room
Room nos permite retornar POJOs personalizados que representen solo las columnas que deseamos retornar si el caso de uso lo amerita.

**Por ejemplo**:

Añade dos columnas más a `ShoppingList` para la fecha de creación y última modificación. Con estas serían ya 5 columnas de nuestra tabla (la propiedad `defaultValue` permite asignar un valor por defecto al campo si no es definido en la inserción).

```java
@ColumnInfo(name = "created_date", defaultValue = "CURRENT_TIMESTAMP")
private final String mCreatedDate;

@ColumnInfo(name = "last_updated", defaultValue = "CURRENT_TIMESTAMP")
private final String mLastUpdated;
```

Ahora crea una nueva clase llamada `ShoppingListForList` y añade solo dos campos: `id` y `name`.

```java
public class ShoppingListForList {
public String id;
public String name;
}
```

Termina asignado esta clase como tipo de retorno y seleccionando las dos columnas objetivo:

```java
@Query("SELECT id, name FROM shopping_list")
LiveData<List<ShoppingListForList>> getAll();

@Query("SELECT id, name FROM shopping_list WHERE category IN(:categories)")
LiveData<List<ShoppingListForList>> getShoppingListsByCategories(List<String> categories);
```

Consultando solo las columnas necesarias optimizas la velocidad de tus consultas.

> aside positive **Nota**: Modifica los métodos del repositorio y **ViewModel** para que acepten este tipo en sus retornos.


### 4. Insertar Registros Parcialmente
También es posible usar POJOs arbitrarios que representen la inserción con tan solo columnas que nos interesen.

**Ejemplo:**

Vamos a crear una entidad llamada `ShoppingListInsert` con solo 3 campos: `id`, `name` y `category`. Donde `category` tomará como valor inicial una de las tres categorías usadas para el ejemplo:

```java
public class ShoppingListInsert {
String id;
String name;
String category = generateCategory();

    public ShoppingListInsert(String id, String name) {
        this.id = id;
        this.name = name;
    }

    public static String generateCategory() {
        String[] categories = new String[]{"Fitness", "Eventos", "Rápidas"};
        return categories[new Random().nextInt(3)];
    }
}
```

Ahora creamos un método para insertar este tipo de objetos llamado `partialInsert()`. Es importante pasarle la propiedad `entity` con la entidad original a la que Room interpretará:

```java
@Insert(onConflict = OnConflictStrategy.IGNORE, entity = ShoppingList.class)
void partialInsert(ShoppingListInsert shoppingList);

@Insert(onConflict = OnConflictStrategy.IGNORE, entity = ShoppingList.class)
void insertShoppingLists(List<ShoppingListInsert> lists);
```

Actualiza también el método `insertShoppingLists()` para la inserción múltiple.

### 5. Actualizar Versión De La Base De Datos
Debido a que nuestro esquema ha cambiado, iremos a `ShoppingListDatabase` y cambia la propiedad `version` por el valor **2**.

Adicionalmente, agrega a la creación de la instancia con el `builder` el método `fallbackToDestructiveMigration()` para eliminar todo el contenido actual de la base de datos y recrearlo con la siguiente versión.

```java
INSTANCE = Room.databaseBuilder(
        context.getApplicationContext(), ShoppingListDatabase.class,
        DATABASE_NAME)
        .addCallback(mRoomCallback)
        .fallbackToDestructiveMigration()
        .build();
```

### 6. Filtrar Listas De Compras
Una vez modificado nuestra capa de datos para satisfacer las características de nuestros bocetos, comenzaremos a crear la interfaz propuesta.

Comencemos con el filtro:

1. Mueve el `RecyclerView` de `activity_main.xml` a un nuevo layout llamado `main_content.xml`. Android Studio puede hacerlo automáticamente si das clic derecho en el componente y presionas **Refactor > Layout**.

    ![refactorlayout](img/refactor_layout.png)

2. Abre el nuevo layout, agrega como nodo raíz un `ConstraintLayout` y sitúa en la parte superior a tres etiquetas `CheckBox`. La solución sería:

    ```
    <?xml version="1.0" encoding="utf-8"?>
    
    <androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="@dimen/normal_padding">
    
        <CheckBox
            android:id="@+id/filter_1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/filter_1"
            app:layout_constraintEnd_toStartOf="@+id/filter_2"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
    
        <CheckBox
            android:id="@+id/filter_2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/filter_2"
            app:layout_constraintEnd_toStartOf="@+id/filter_3"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toEndOf="@+id/filter_1"
            app:layout_constraintTop_toTopOf="parent" />
    
        <CheckBox
            android:id="@+id/filter_3"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/filter_3"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toEndOf="@+id/filter_2"
            app:layout_constraintTop_toTopOf="parent" />
    
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/list"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_marginTop="@dimen/normal_padding"
            app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/filter_2"
            tools:listitem="@layout/shopping_list_item"
            tools:showIn="@layout/activity_main" />
    
    </androidx.constraintlayout.widget.ConstraintLayout>
    ```

3. Procesa las categorías seleccionadas agregando al `ViewModel` un `LiveData` para ellas. Complementariamente, provee un método para añadirlas y otro para removerlas:

    ```java
    public class ShoppingListViewModel extends AndroidViewModel {
    
        // Filtros observados
        private final MutableLiveData<List<String>> mCategories
                = new MutableLiveData<>(new ArrayList<>());
    
    
        // Filtros
        private final List<String> mFilters = new ArrayList<>();
        
    
        public void addFilter(String category) {
            mFilters.add(category);
            mCategories.setValue(mFilters);
        }
    
        public void removeFilter(String category) {
            mFilters.remove(category);
            mCategories.setValue(mFilters);
        }
    }
    ```

4. Usaremos una transformación `switchMap()` para obtener las listas de compras por categorías en el constructor del `ViewModel`. Si no existen categorías marcadas entonces obtenemos todos los registros:

    ```java
    public ShoppingListViewModel(@NonNull Application application) {
    super(application);
    mRepository = new ShoppingListRepository(application);
    
            // Obtener listas de compras por categorías
            mShoppingLists = Transformations.switchMap(
                    mCategories,
                    categories -> {
                        if (categories.isEmpty()) {
                            return mRepository.getShoppingLists();
                        } else {
                            return mRepository.getShoppingListsWithCategories(categories);
                        }
                    }
            );
    }
    ```

5. Por último, actualizamos `mFilters` desde `MainActivity` a través del método `setupFilters()`. Aquí conseguiremos las referencias de los **CheckBoxes** y les añadimos una escucha. Esta añade o elimina los filtros dependiendo de su estado.

    ```java
    private void setupFilters() {
    mFilters = new ArrayList<>();
    mFilters.add(findViewById(R.id.filter_1));
    mFilters.add(findViewById(R.id.filter_2));
    mFilters.add(findViewById(R.id.filter_3));
    
            // Definir escucha de filtros
            CompoundButton.OnCheckedChangeListener listener = (compoundButton, checked) -> {
                String category = compoundButton.getText().toString();
                if (checked) {
                    mViewModel.addFilter(category);
                } else {
                    mViewModel.removeFilter(category);
                }
            };
    
            // Setear escucha
            for (CheckBox filter : mFilters) {
                filter.setOnCheckedChangeListener(listener);
            }
    }
    ```

Si ejecutas el proyecto podrás filtrar las listas por la categoría asignada:

<img src="img/sin-filtro-room-app-616x1300.png" width="300" alt="sin_filtro"> <img src="img/filtro-checkbox-android-616x1300.png" width="300" alt="filtro_checkbox">

### 7. Editar Listas De Compras
Por el momento la pantalla de edición estará vacía. El único dato que desplegaremos será el nombre en la [**Toolbar**](https://www.develou.com/toolbar-en-android-creacion-de-action-bar-en-material-design/).

Veamos cómo conseguirlo:

#### 1. Crear Actividad De Edición
Añade una nueva actividad yendo a **File > New > Activity > Empty Activity** y nombrala `EditShoppingListActivity`. Seguido, habilita la navegación hacia arriba:

```java
public class EditShoppingListActivity extends AppCompatActivity {

    public static final String EXTRA_SHOPPING_LIST_ID = "com.develou.shoppinglist.shoppingListId";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_edit_shopping_list);
        
        // Obtener id de la lista de compras
        String id = getIntent().getStringExtra(EXTRA_SHOPPING_LIST_ID);
        
        setupActionBar();
    }

    private void setupActionBar() {
        ActionBar actionBar = getSupportActionBar();

        actionBar.setDisplayHomeAsUpEnabled(true);
    }

    @Override
    public boolean onSupportNavigateUp() {
        onBackPressed();
        return true;
    }
}
```

Recuerda que recibiremos el id de la lista de compras, por lo que es necesario extraerlo desde el extra.

#### 2. Añadir Escucha De Ítems Al Adaptador
Hasta el momento nuestro **adaptador** no reaccionaba a los eventos de clics sobre sus ítems, así que añadiremos una interfaz que se encargue de esta responsabilidad.
    
Esto implica:

 - Añadir una interfaz de escucha al adaptador.
 - Procesar el **clic** sobre cada ítem en el `ViewHolder`.
 - Añadir un método para asignar la escucha.
 - Ubicar como clase interna al `ViewHolder`.

Al codificar todas las características mencionadas tendrás:

```java
public class ShoppingListAdapter
extends RecyclerView.Adapter<ShoppingListAdapter.ShoppingListViewHolder> {

    private List<ShoppingListForList> mShoppingLists;
    private ItemListener mItemListener;

    @NonNull
    @Override
    public ShoppingListViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        return new ShoppingListViewHolder(
                LayoutInflater.from(parent.getContext())
                .inflate(R.layout.shopping_list_item, parent, false)
        );
    }

    @Override
    public void onBindViewHolder(@NonNull ShoppingListViewHolder holder, int position) {
        ShoppingListForList item = mShoppingLists.get(position);
        holder.bind(item);
    }

    @Override
    public int getItemCount() {
        return mShoppingLists == null ? 0 : mShoppingLists.size();
    }

    public void setItems(List<ShoppingListForList> items) {
        mShoppingLists = items;
        notifyDataSetChanged();
    }

    public void setItemListener(ItemListener listener) {
        mItemListener = listener;
    }

    interface ItemListener {
        void onClick(ShoppingListForList shoppingList);
    }

    public class ShoppingListViewHolder extends RecyclerView.ViewHolder {
        private final TextView mNameText;

        public ShoppingListViewHolder(@NonNull View itemView) {
            super(itemView);
            mNameText = itemView.findViewById(R.id.name);
            itemView.setOnClickListener(view -> {
                if (mItemListener != null) {
                    ShoppingListForList clickedItem = mShoppingLists.get(getAdapterPosition());
                    mItemListener.onClick(clickedItem);
                }
            });
        }

        public void bind(ShoppingListForList item) {
            mNameText.setText(item.name);
        }
    }
}
```

#### 3 Iniciar Actividad De Edición
Añade una escucha al adaptador en `MainActivity`. La idea es enviar el `Id` de la lista de compras en el `Intent` explícito con la finalidad de procesarlo en la actividad de edición.

```java
private void setupList() {
    mList = findViewById(R.id.list);
    mAdapter = new ShoppingListAdapter();
    mList.setAdapter(mAdapter);
    
    // Asignar escucha de ítems
    mAdapter.setItemListener(this::editShoppingList);
    
    // Observar cambios de listas de compras
    mViewModel.getShoppingLists().observe(this, mAdapter::setItems);
}


private void editShoppingList(ShoppingListForList shoppingList) {
    Intent intent = new Intent(MainActivity.this,
    EditShoppingListActivity.class);
    intent.putExtra(EditShoppingListActivity.EXTRA_SHOPPING_LIST_ID,
    shoppingList.id);
    startActivity(intent);
}
```
Crea un nuevo método en el repositorio para cargar una lista de compras por `ID`:

```java
public LiveData<ShoppingList> getShoppingList(String id){
    return mShoppingListDao.getShoppingList(id);
}
```

#### 4 Crear ViewModel Para Edición
Luego, crea la clase `EditShoppingListViewModel` y:

 - Hazla extender de `AndroidViewModel`.
- Añade un `LiveData` para el `ID` de la lista.
- Añade un `LiveData` para la lista a editar. Relaciónala con el `ID` a través de una transformación `switchMap()`.
- Agrégale un método para cargar la lista con el repositorio.

El código sería el siguiente:

```java
public class EditShoppingListViewModel extends AndroidViewModel {

    private final ShoppingListRepository mRepository;

    private final MutableLiveData<String> mShoppingListId = new MutableLiveData<>();

    private final LiveData<ShoppingList> mShoppingList;

    public EditShoppingListViewModel(@NonNull Application application) {
        super(application);
        mRepository = new ShoppingListRepository(application);
        mShoppingList = Transformations.switchMap(
                mShoppingListId,
                mRepository::getShoppingList
        );
    }

    public void start(String id){
        if(id.equals(mShoppingListId.getValue())){
            return;
        }
        mShoppingListId.setValue(id);
    }

    public LiveData<ShoppingList> getShoppingList() {
        return mShoppingList;
    }
}
```

Para finalizar, actualizamos la interfaz al observar `mShoppingList` en la actividad. Cuando se cargue la lista de compras a editar cambiamos el título de la `Toolbar` por el nombre del registro:

```java
private void setupActionBar() {
ActionBar actionBar = getSupportActionBar();

        actionBar.setDisplayHomeAsUpEnabled(true);

        mViewModel.getShoppingList().observe(this,
                shoppingList -> actionBar.setTitle(shoppingList.getName())
        );
}
```

Si ejecutas el proyecto y, como ejemplo, seleccionas la **«Lista 4»** deberías ver lo siguiente:

<img src="img/edicion-lista-de-compras-616x1300.png" width="300" alt="Edición_lista_compras">

