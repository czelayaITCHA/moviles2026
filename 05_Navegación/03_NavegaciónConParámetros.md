# Navegación con parámetros utilizando `navArgument` en Jetpack Compose

### **Objetivo:**
El objetivo de este material es aprender cómo utilizar `navArgument` en Jetpack Compose para pasar parámetros entre pantallas durante la navegación. Esta es una técnica importante que se usa comúnmente en proyectos reales.

### **Requisitos previos:**
- Conocimiento básico de Jetpack Compose (como las funciones `Composable` y el uso de `Modifier`)
- Familiaridad con la navegación usando `NavController`

### **Contenido de aprendizaje:**
- Navegación con parámetros utilizando `navArgument`
- Especificar el tipo de parámetro con `NavType`

---

## **1. Sintaxis básica y pasos para implementar la navegación con `navArgument`**

Primero, aprenderemos cómo pasar parámetros de una pantalla a otra utilizando `navArgument`. En este ejemplo, se pasará un parámetro de texto desde una pantalla A a una pantalla B.

### Pasos para implementar
1. **Define una `ruta` con parámetros para una función `composable` en NavHost**.
   <br>
   Define una pantalla (componente) utilizando la función `composable`. En este caso, especifica el nombre del parámetro encerrándolo con {} en la `route`. Pasa una lista de pares nombre/tipo de parámetro (`NavType`) a `navArgument` y especifica el tipo de datos de cada parámetro.

2. **Define la lógica de obtención de parámetros desde `backStackEntry.arguments` en el bloque de la función `composable`**.
   <br>
   Definir la lógica en el bloque de la función `composable` para obtener los valores de los parámetros desde `backStackEntry.arguments` y pasarlos a la pantalla a la que se va a realizar la transició

3. **Pasar argumentos al pasar de una pantalla a otra**.
   <br>
   Utiliza el método `navigate()` del `NavController` en el momento de la transición para incrustar parámetros en `route`.

### **Ejemplo de código:**
Este código muestra un ejemplo sencillo de navegación y paso de parámetros utilizando Jetpack Compose. La función `NavManager` gestiona toda la navegación de la aplicación, donde se ingresa un nombre en la pantalla principal (`HomeScreen`) y se pasa a la pantalla de detalles (`DetailsScreen`) para ser mostrado. A continuación, se explica cada parte del código:

#### 1. Función `NavManager`
```kotlin
@Composable
fun NavManager() {
    val navController = rememberNavController()
    NavHost(navController = navController, startDestination = "home") {

        composable("home") {
            HomeScreen(navController)
        }
        composable(
            route = "details/{nombre}",
            arguments = listOf(navArgument("nombre") { type = NavType.StringType })
        ) { backStackEntry ->
            val nombre = backStackEntry.arguments?.getString("nombre") ?: ""
            DetailsScreen(nombre)
        }
    }
}
```
- **Especificar el parámetro y tipo de dado dentro del método `composable`.**
  - **`composable(route = "details/{nombre}")`**: La ruta `"details/{nombre}"` está asociada a la pantalla `DetailsScreen`, que recibe un parámetro llamado `nombre`.
  - **`navArgument("nombre")`**: Define el argumento `nombre` como de tipo cadena (`StringType`).

- **Definir la lógica de recuperación de parámetros utilizando el objeto `backStackEntry`.**
  - **`backStackEntry`**: Contiene los argumentos de navegación. Utilizamos `backStackEntry.arguments` para obtener el parámetro `nombre` pasado desde la pantalla anterior.
  - `getString("nombre")`: Este método recupera el valor del argumento asociado con la clave `"nombre"`, que es el parámetro que se pasó al navegar. En este caso, esperamos que sea una cadena de texto.
  - **`DetailsScreen(nombre)`**: Llama a la pantalla de detalles y le pasa el valor de `nombre` para que sea mostrado.

> [!NOTE]
> Operadores para seguridad nula
> - `?.`: Es el operador de llamada segura en Kotlin. Se utiliza para evitar un error en caso de que `arguments` sea `null`. Si `arguments` no es `null`, se llama a `getString("nombre")`; de lo contrario, el valor será `null` sin causar una excepción.
> - `?: ""`: Este es el operador Elvis (`?:`) que establece un valor predeterminado si la expresión de la izquierda es `null`. En este caso, si no se encuentra el valor de `"nombre"` o si es `null`, el código asigna una cadena vacía `""` a `nombre`.

#### 2. Función `HomeScreen`
```kotlin
@Composable
fun HomeScreen(navController: NavController) {
    var nombre by remember { mutableStateOf("") }

    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        OutlinedTextField(
            value = nombre,
            onValueChange = {nombre = it},
            label = { Text(text = "Nombre")},
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp)
                .padding(bottom = 12.dp)
        )
        Button(onClick = { navController.navigate("details/$nombre") }) {
            Text("Go to Details")
        }
    }
}
```
- **Pasar los valores de los argumentos desde la pantalla de origen como parte de la ruta.**
  - **`Button(onClick)`**: Al hacer clic en el botón, se navega a la pantalla `DetailsScreen` utilizando la función `navController.navigate("details/$nombre")`. El nombre ingresado es pasado como parte de la ruta.

#### 3. Función `DetailsScreen`
```kotlin
@Composable
fun DetailsScreen(nombre: String) {
    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("Nombre: $nombre")
    }
}
```
- **`DetailsScreen(nombre: String)`**: Esta pantalla recibe el parámetro `nombre` desde la pantalla `HomeScreen` y lo muestra en un texto.

---

## **2. Pasar múltiples parámetros**
Ahora aprenderemos cómo pasar múltiples parámetros entre pantallas. En este caso, pasaremos el nombre y el edad.

### Repaso: Pasos para implementar
Básicamente, se pueden definir múltiples parámetros utilizando el mismo paso de implementación que cuando sólo hay un parámetro durante la navegación.
1. **Define una `ruta` con parámetros para una función `composable` en NavHost**.
2. **Define la lógica de obtención de parámetros desde `backStackEntry.arguments` en el bloque de la función `composable`**.
3. **Pasar argumentos al pasar de una pantalla a otra**.

### **Ejemplo de código:**
#### 1. Función `NavManager`
```diff
@Composable
fun NavManager() {
    val navController = rememberNavController()
    NavHost(navController = navController, startDestination = "home") {

        composable("home") {
            HomeScreen(navController)
        }
        composable(
+            route = "details/{nombre}/{edad}",
            arguments = listOf(
                navArgument("nombre") { type = NavType.StringType },
+                navArgument("edad") { type = NavType.IntType }
            )
        ) { backStackEntry ->
            val nombre = backStackEntry.arguments?.getString("nombre") ?: ""
+            val edad = backStackEntry.arguments?.getInt("edad") ?: 0
+            DetailsScreen(nombre, edad)
        }
    }
}
```
#### 2. Función `HomeScreen`
```diff
@Composable
fun HomeScreen(navController: NavController) {
    var nombre by remember { mutableStateOf("") }
+    var edad by remember { mutableStateOf("") }

    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        OutlinedTextField(
            value = nombre,
            onValueChange = {nombre = it},
            label = { Text(text = "Nombre")},
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp)
                .padding(bottom = 12.dp)
        )
+        OutlinedTextField(
+            value = edad,
+            onValueChange = {edad = it},
+            label = { Text(text = "Edad")},
+            modifier = Modifier
+                .fillMaxWidth()
+                .padding(horizontal = 16.dp)
+                .padding(bottom = 12.dp)
+        )

+        Button(onClick = { navController.navigate("details/$nombre/$edad") }) {
            Text("Go to Details")
        }
    }
}
```
#### 3. Función `DetailsScreen`
```diff
@Composable
+ fun DetailsScreen(nombre: String, edad: Int) {
    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("Nombre: $nombre")
+        Text("Edad: $edad")
    }
}
```

### **Puntos clave:**
- Para pasar múltiples parámetros, especificamos `"details/{nombre}/{edad}"` en la ruta.
- Usamos `navArgument` para definir el tipo de cada parámetro.

## Ejercicios
### Ejercicio1
1. Crear un archivo Kotlin `SaludoScreen` en el paquete `components`.
2. `SaludoScreen`Pegue el siguiente código en `SaludoScreen`.
   ```kotlin
   @Composable
   fun SaludoScreen(nombre: String, edad: Int) {
       Column(
           modifier = Modifier.fillMaxSize(),
           verticalArrangement = Arrangement.Center,
           horizontalAlignment = Alignment.CenterHorizontally
       ) {
           Text("Hola, soy $nombre. Tengo $edad años.")
       }
   }
   ```
3. Modifique el código para que los valores introducidos en la `HomeScreen` se muestren en la `SaludoScreen`, como se muestra en el siguiente diagrama.
   - Dentro de `NavHost` de `NavManager`, añade una llamada a la función `composable` para definir la navegación a la `SaludoScreen`.
   - Añade un botón a la `HomeScreen` para navegar a la `SaludoScreen`.

   --falta imagen

### Ejercicio2
1. Modifique el código para que el apellido introducido en la `HomeScreen` se muestre en la `SaludoScreen`, como se muestra en el siguiente diagrama.
   - Añadir un campo de apellido a la `HomeScreen`.
   - Dentro de `NavHost` de `NavManager, modificar la definición de navegación a la `SaludoScreen`.
   - En `HomeScreen`, modificar el botón para navegar a la `SaludoScreen`.
   - Añade el apellido al parámetro `SaludoScreen` y modifícalo para mostrarlo.

--falta imagen
