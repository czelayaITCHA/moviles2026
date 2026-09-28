# Fundamentos de `Navigation`: Cómo navegar entre pantallas con Compose
## Descripción general de `Navigation` en Jetpack Compose
¿Por qué es importante `Navigation`?
- **Función de `Navigation`**: Gestiona las transiciones entre pantallas y permite que el usuario navegue entre diferentes pantallas dentro de la aplicación.
- **Gestión de la UI**: Se utiliza para controlar cómo se organizan y transitan las pantallas dentro de la aplicación.
- **Diferencia en Jetpack Compose**: A diferencia del diseño XML tradicional, en Jetpack Compose las interfaces de usuario se definen como funciones Composable.

---
## **Componentes principales de `Navigation`**
- **NavController**: Controlador que gestiona las transiciones entre pantallas y el backstack.
  - Ejemplo:
    ```kotlin
    val navController = rememberNavController()
    ```
- **NavHost**: El lugar donde se configura la navegación. Define qué pantalla se muestra.
  - Ejemplo:
    ```kotlin
    NavHost(navController = navController, startDestination = "home")
    ```
- **composable**: Función Composable para definir cada pantalla.
  - Ejemplo:
    ```kotlin
    composable("home") { HomeScreen(navController) }
    ```

---
## Ejemplos: Transición simple entre pantallas
Vamos a ver un ejemplo de transición de `HomeScreen` a `DetailsScreen`.
1. Crea un proyecto llamado `EjemploNavigation`.
2. Eliminar la función por defecto `Greeting` y su código de llamada.
3. Copia y pega el siguiente código.
```kotlin
@Composable
fun MyApp() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = "home") {
        composable("home") {
            HomeScreen(navController)
        }
        composable("details") {
            DetailsScreen()
        }
    }
}

@Composable
fun HomeScreen(navController: NavController) {
    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Button(onClick = { navController.navigate("details") }) {
            Text("Go to Details")
        }
    }
}

@Composable
fun DetailsScreen() {
    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("This is the Details Screen")
    }
}
```
4. `rememberNavController()` te dará un error, así que pásate por encima, teclea `Alt` y `Enter` y añade una dependencia.
![image](https://github.com/user-attachments/assets/fa42a2b0-c58a-427c-bcaa-67c0c3465d4f)
5. Importe los elementos que faltan para eliminar errores.
6. define `MyApp()` en el bloque del `setContent` de la función `onCreate` para ejecutar la app.

### Explicación.
#### 1. Función `MyApp`
- **`rememberNavController()`**
  - **Punto:** `rememberNavController()` crea un `NavController` que gestiona la navegación. Este controlador se utiliza para moverse entre diferentes pantallas.
  - **Analogía:** El `NavController` es como un mapa dentro de la aplicación que indica a dónde se debe ir en cada pantalla.
- **`NavHost`**
  - **Punto:** `NavHost` define la estructura de navegación de la aplicación. Especifica cuál pantalla se mostrará primero y a cuáles pantallas se puede navegar.
  - **`startDestination`**
    - **Punto:** `startDestination` indica la pantalla (ruta) que se mostrará primero en la aplicación. En este caso, `"home"` es la primera pantalla.
- **`composable`**
  - **Punto:** La función `composable` define los componentes (pantallas) que se pueden navegar. En este caso, se pueden navegar `HomeScreen` y `DetailsScreen`.

#### 2. Función `HomeScreen`
- **`Button`**
  - **Punto:** `Button` crea un botón que los usuarios pueden hacer clic. 
  - **`onClick = { navController.navigate("details") }`**
    - **Punto:** Cuando se hace clic en el botón, se utiliza `navController` para navegar a la pantalla `"details"`.

#### Resumen
- **NavController:** Objeto que gestiona la navegación entre pantallas.
- **NavHost:** Lugar donde se definen las reglas de navegación de la aplicación.
- **Funciones Composable:** Funciones que definen elementos de la interfaz de usuario.
- **Button:** Elemento interactivo que permite a los usuarios realizar acciones.

---
## ¿Qué es una `Route` de `Navigation` ?
`route` es la "ruta" que se utiliza para identificar una pantalla (Composable) al navegar dentro de la aplicación. Cada pantalla (Composable) es identificada por una `route` única, lo que permite especificar a qué pantalla se debe navegar en la aplicación.

### Uso básico

1. **Definición**  
   Cada pantalla (Composable) se registra dentro del `NavHost` usando la función `composable`, donde se identifica mediante una `route`.

   ```kotlin
   NavHost(navController = navController, startDestination = "home") {
       composable("home") { HomeScreen(navController) }
       composable("details") { DetailsScreen() }
   }
   ```

   En el ejemplo anterior, `"home"` y `"details"` son las `routes` que identifican a cada pantalla.

2. **Navegación**  
   Se utiliza el `NavController` para navegar a la `route` especificada.

   ```kotlin
   Button(onClick = { navController.navigate("details") }) {
       Text("Ir a Detalles")
   }
   ```

   Este código navega a la `route` `"details"` cuando se presiona el botón.

### Resumen
- **Route** es la ruta que identifica de manera única una pantalla.
- **Definir una Route** se hace dentro del `NavHost` con la función `composable`.
- **Navegar** se realiza usando el `NavController` para moverse a una `route`.


## Ejercicio

El código inicial que figura a continuación está incompleto. Por favor, corrija las secciones comentadas para completarlo.

```kotlin
@Composable
fun MyApp2() {
    val navController = /* TODO */

    NavHost(navController = navController, startDestination = "home2") {
        composable("home2") {
            /* TODO */
        }
        composable("details2") {
            /* TODO */
        }
    }
}

@Composable
fun HomeScreen2(navController: NavController) {
    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Button(onClick = { /* TODO */ }) {
            Text("Go to Details")
        }
    }
}

@Composable
fun DetailsScreen2() {
    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("This is the Details Screen")
    }
}
```
