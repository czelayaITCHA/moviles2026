# Transiciones de pantalla con Backstack

## Consejo: Separar los paquetes para mejorar la legibilidad
Si la cantidad de código en MainActivity es demasiado grande, la legibilidad disminuirá.
Para una mejor visibilidad, crea paquetes y mueve el código a archivos separados.

1. Crear nuevo proyecto con el nombre `NavigationApp`
2. Eliminar funcion composable `Greeting` y su llamada en MainActivity.kt
3. Pega el siguiente código inicial en `MainActivity.kt` y define una llamada a `NavManager` en `onCreate`.
   ```kotlin
   @Composable
   fun NavManager() {
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
5. Crear package `components` dentro del package principal del proyecto
   ![image](https://github.com/user-attachments/assets/f2e86904-e07f-4246-aa20-a243e4ddb446)
6. Crear unos archivos dentro del package `components`
- Primero creamos el archivo `HomeScreen`
  ![image](https://github.com/user-attachments/assets/22516a52-10d3-47f2-8859-b1cb808de121)
  ![image](https://github.com/user-attachments/assets/11bfb61e-0021-4858-9eba-286e20753515)

- Segundo creamos el archivo `DetailsScreen`
  ![image](https://github.com/user-attachments/assets/ee397418-7563-4f2b-897d-b0139a81d142)
  ![image](https://github.com/user-attachments/assets/98143053-4e0b-4c64-b239-f91108a9d51b)

6. Mueva el código fuente de las funciones componibles en `MainActivity.kt` al archivo que ha creado.
   ![image](https://github.com/user-attachments/assets/a73b3d35-82eb-4c1e-9721-0efc40193e4e)
   ![image](https://github.com/user-attachments/assets/b0818d10-d03c-4b14-a204-908f7f8eddbc)


7. Crear package `navigation` dentro del package principal del proyecto
   ![image](https://github.com/user-attachments/assets/eb4f1984-7d98-48e5-992a-ca45625495c8)

8. Crear un archivo `NavManager` dentro del package `navigation`
   ![image](https://github.com/user-attachments/assets/616d5ac6-8995-4343-addc-f3cd38a6f237)

9. Mueva el código fuente de la funcion `NavManager` en `MainActivity.kt` al archivo `NavManager.kt` que ha creado.
   ![image](https://github.com/user-attachments/assets/035bec1e-c87e-4a69-995f-62c8df6e2f27)

10. Ejecutar la aplicación.

## Conceptos Básicos del `Backstack`

### 2.1 ¿Qué es el Backstack?

- **Definición del backstack:**  
  - El backstack es una estructura de pila que realiza un seguimiento de las pantallas por las que ha pasado el usuario durante la navegación.
  - Al presionar el botón de retroceso, se retorna a la pantalla anterior en el stack.

- **Cuándo usar el backstack:**  
  - Cuando se desea regresar desde una pantalla de detalles y mantener el estado de la pantalla anterior (como una lista).
  - Cuando se navega a través de varias pantallas y se necesita gestionar el historial de navegación.

### 2.2 Ejemplo de implementación

- Usar `popBackStack()` del `NavController` para remover la pantalla actual del backstack y regresar a la pantalla anterior.

```kotlin
Button(onClick = { navController.popBackStack() }) {
    Text("Volver")
}
```

- **Nota:**  
  - `popBackStack()` no siempre tiene éxito, especialmente si estás en la pantalla inicial.

### 2.3 Ejercicio: Añadir un botón de retroceso a la `DetailsScreen`.
- Abajo del `Text` en `DetailsScreen`, añade el boton del ejemplo introducido.
- Añade `navController: NavController` como parámetro de `DetailsScreen`.
- Dentro de `NavManager`, añade un argumento a la llamada a `DetailsScreen`.
