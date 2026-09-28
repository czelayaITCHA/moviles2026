# Gestión del estado

## **1. Explicación de State (Estado)**

El `State` en Jetpack Compose es la forma de mantener y gestionar datos dinámicos en la UI. Siempre que el estado cambie, la UI se vuelve a renderizar automáticamente para reflejar los cambios.
Por ejemplo, el texto cambia cuando se presiona un botón o el aspecto de un componente cambia cuando se marca una casilla de verificación, todo esto en función del estado.

### **Sintaxis básica de State**
```kotlin
var text by remember { mutableStateOf("") }  // El estado inicial es una cadena vacía
```
- `remember` y `mutableStateOf` se usan juntos para gestionar el estado.
  - `remember` se utiliza para almacenar un valor que debe conservarse incluso cuando el componente se recompone.
  - `mutableStateOf` crea un dato que se tratará como un estado. Cuando este dato cambia, la UI se vuelve a renderizar automáticamente.
  - Se puede utilizar el operador delegado `by` de Kotlin para hacer que el acceso y la modificación del estado sean más simples y claros.
- Cuando el estado cambia, se redibuja automáticamente la UI que depende de ese estado.

En este ejemplo, cuando se cambia el valor de `text`, se redibuja el componente que hace referencia a `text`.

### Comparación con `React.js`
Este código muestra cómo gestionar el estado en React.js, similar al uso de `remember` y `mutableStateOf` en Jetpack Compose.
```javascript
// Definir el estado con useState. El estado inicial es una cadena vacía
const [text, setText] = useState("");

```
- `useState("")`: En React, utilizamos `useState` para definir el estado `text`, que inicialmente es una cadena vacía.
- `setText`: Esta es la función que se usa para actualizar el estado `text`. Cuando llamamos a `setText`, React vuelve a renderizar el componente con el nuevo valor del estado.


---

## **2. Ejemplos con `OutlinedButton` y `OutlinedTextField`**

### **Ejemplo: OutlinedButton para mostrar un mensaje al hacer clic**

Ahora, crearemos un ejemplo donde se muestra un mensaje cuando el botón es clicado, permitiendo verificar el funcionamiento de `onClick`.

```kotlin
@Composable
fun OutlinedButtonMensaje() {
    var mensaje by remember { mutableStateOf("Aún no has hecho clic") }

    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center,
        modifier = Modifier.fillMaxSize()
    ) {
        OutlinedButton(
            onClick = {
                mensaje = "¡Has hecho clic en el botón!"
            }
        ) {
            Text(text = "Haz clic aquí")
        }
        Spacer(modifier = Modifier.height(16.dp))
        Text(text = mensaje)
    }
}
```

**Explicación**:  
- Utilizamos `remember` y `mutableStateOf` para definir una variable `mensaje` que retiene el estado. Cuando se hace clic en el botón, el mensaje cambia.
- El texto debajo del botón se actualiza dinámicamente según el estado.

### **Ejemplo: OutlinedButton para añadir un contador de clics**

A continuación, implementamos una función que cuenta cuántas veces se ha hecho clic en el botón.

```kotlin
@Composable
fun OutlinedButtonContador() {
    var count by remember { mutableStateOf(0) }

    Column(
        modifier = Modifier.padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        OutlinedButton(
            onClick = {
                count++
            }
        ) {
            Text(text = "Haz clic para contar")
        }
        Spacer(modifier = Modifier.height(16.dp))
        Text(text = "Cantidad de clics: $count")
    }
}
```

**Explicación**:  
- Cada vez que se hace clic en el botón, el valor de `count` se incrementa en 1. `remember` y `mutableStateOf` se utilizan para mantener el estado del contador.
- El texto que muestra el número de clics se actualiza dinámicamente.

### **Ejemplo: Crear un OutlinedTextField básico**
Crea un campo de texto simple utilizando `OutlinedTextField` y muestra en tiempo real el texto que el usuario ingresa en la pantalla.

```kotlin
@Composable
fun BasicTextFieldExample() {
    var text by remember { mutableStateOf("") }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.Center
    ) {
        // OutlinedTextField para capturar la entrada del usuario
        OutlinedTextField(
            value = text,        // El estado actual del texto
            onValueChange = { newText -> text = newText }, // Qué hacer cuando el texto cambia
            label = { Text("Introduce aquí") },   // Etiqueta del campo de entrada
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(16.dp))

        // Mostrar el texto ingresado en tiempo real
        Text(text = "Texto ingresado: $text")
    }
}
```

**Explicación**:
- El `OutlinedTextField` captura la entrada del usuario y, con la función `onValueChange`, el valor de `text` se actualiza cada vez que el usuario escribe.
- El componente `Text` muestra el texto ingresado en tiempo real.

### **Ejemplo: Restablecer el contenido del campo de texto**
Crea un campo de texto y un botón de "Restablecer". Al hacer clic en el botón, el contenido del campo de texto debe quedar vacío.

```kotlin
@Composable
fun ResetTextFieldExample() {
    var text by remember { mutableStateOf("") }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.Center
    ) {
        OutlinedTextField(
            value = text,
            onValueChange = { newText -> text = newText },
            label = { Text("Ingresa texto") },
            modifier = Modifier.fillMaxWidth()
        )
        Spacer(modifier = Modifier.height(16.dp))
        // Botón para restablecer el contenido del campo de texto
        OutlinedButton(
            onClick = { text = "" }, // Asigna una cadena vacía.
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Restablecer")
        }
    }
}
```

**Explicación**:
- Al hacer clic en el botón, la variable `text` se vacía, lo que restablece el contenido del `OutlinedTextField`.

### **Ejemplo: Múltiples campos**
Crea dos `OutlinedTextField` donde el usuario pueda ingresar su nombre y edad. Al presionar el botón de "Enviar", debería mostrarse en la pantalla: "[Nombre] tiene [Edad] años".

```kotlin
@Composable
fun MultipleFieldsExample() {
    var name by remember { mutableStateOf("") }
    var age by remember { mutableStateOf("") }
    var result by remember { mutableStateOf("") }

    Column(
        modifier = Modifier.padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        OutlinedTextField(
            value = name,
            onValueChange = { name = it },
            label = { Text("Nombre") }
        )
        Spacer(modifier = Modifier.height(16.dp))
        OutlinedTextField(
            value = age,
            onValueChange = { age = it },
            label = { Text("Edad") },
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number)
        )
        Spacer(modifier = Modifier.height(16.dp))
        OutlinedButton(onClick = {
            result = "$name tiene $age años"
        }) {
            Text("Enviar")
        }
        Spacer(modifier = Modifier.height(16.dp))
        Text(result)
    }
}
```
**Explicación**:
- `name` y `age` se utilizan para almacenar el valor ingresado en cada campo de texto.
- Al hacer clic en el botón, se muestra el resultado basado en el nombre y la edad ingresados.

### **Ejemplo : Habilitar y deshabilitar un campo de texto**

Crea un botón que habilite o deshabilite un `OutlinedTextField` al hacer clic. Inicialmente, el campo debe estar habilitado.

```kotlin
@Composable
fun ToggleTextFieldEnabledExample() {
    var text by remember { mutableStateOf("") }
    var isEnabled by remember { mutableStateOf(true) }

    Column(
        modifier = Modifier.padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        OutlinedTextField(
            value = text,
            onValueChange = { text = it },
            label = { Text("Campo de texto") },
            enabled = isEnabled
        )
        Spacer(modifier = Modifier.height(16.dp))
        OutlinedButton(onClick = { isEnabled = !isEnabled }) {
            Text(if (isEnabled) "Deshabilitar" else "Habilitar")
        }
    }
}
```

**Explicación**:
- El estado `isEnabled` se alterna cada vez que se hace clic en el botón, lo que habilita o deshabilita el `OutlinedTextField`.
- El texto que aparece en el botón cambia según el valor de `isEnabled`.

---

## **4. Ejercicios**

### **Ejercicio 1: Contador de clics**
**Problema**: Crea un botón que cuente el número de clics realizados. Cada vez que se hace clic, se debe mostrar un mensaje indicando el número de clics.

**Ejemplo de diseño:**
<br>
![image](https://github.com/user-attachments/assets/dcee59a0-dc14-41cc-bf77-2d654c26fd80)
<!--
<details>
  <summary>Ejemplo de solución</summary>

  ```kotlin
   @Composable
   fun ClickCountExample() {
       var count by remember { mutableStateOf(0) }

       Column(
           modifier = Modifier.padding(16.dp),
           horizontalAlignment = Alignment.CenterHorizontally
       ) {
           OutlinedButton(onClick = { count++ }) {
               Text("Clic")
           }
           Spacer(modifier = Modifier.height(16.dp))
           Text("El botón ha sido clickeado $count veces")
       }
   }
   ```
   **Explicación**:
   - Cada vez que se hace clic en el botón, el valor de `count` se incrementa y el número de clics se actualiza en la pantalla.
</details>
-->

### **Ejercicio 2: OutlinedTextField de nombre**
**Problema**: Añade una etiqueta al `OutlinedTextField` que diga "Por favor, ingrese su nombre" y muestra el texto ingresado debajo.

**Ejemplo de diseño:**
<br>
![image](https://github.com/user-attachments/assets/9f8096b3-7bec-4b98-9e20-0257242e58b0)

<!--
<details>
  <summary>Ejemplo de solución</summary>

   ```kotlin
   @Composable
   fun LabeledTextFieldExample() {
       var name by remember { mutableStateOf("") }

       Column(
           modifier = Modifier
               .fillMaxSize()
               .padding(16.dp),
           verticalArrangement = Arrangement.Center
       ) {
           // OutlinedTextField con etiqueta
           OutlinedTextField(
               value = name,
               onValueChange = { newName -> name = newName },
               label = { Text("Por favor, ingrese su nombre") },
               modifier = Modifier.fillMaxWidth()
           )

           Spacer(modifier = Modifier.height(16.dp))

           // Mostrar el nombre ingresado
           Text(text = "Nombre ingresado: $name")
       }
   }
   ```

   **Explicación**:
   - Se añade una etiqueta al `OutlinedTextField` para dar indicaciones al usuario.
   - El nombre ingresado se muestra en el componente `Text` debajo del campo de texto.
</details>
-->
### **Ejercicio 3: Contar caracteres ingresados y mostrar el total**
**Problema**: Crea una aplicación que cuente los caracteres ingresados en un `OutlinedTextField` y los muestre en tiempo real.

**Ejemplo de diseño:**
<br>
![image](https://github.com/user-attachments/assets/91edf3fc-fe2e-4c63-86d3-ae342fd52718)

<!--
<details>
  <summary>Ejemplo de solución</summary>

   ```kotlin
   @Composable
   fun CountCharactersExample() {
       var text by remember { mutableStateOf("") }

       Column(
           modifier = Modifier.padding(16.dp),
           horizontalAlignment = Alignment.CenterHorizontally
       ) {
           OutlinedTextField(
               value = text,
               onValueChange = { text = it },
               label = { Text("Ingresa texto") }
           )
           Spacer(modifier = Modifier.height(16.dp))
           Text("Número de caracteres: ${text.length}")
       }
   }
   ```

   **Explicación**:  
   - `text.length` se usa para contar los caracteres ingresados en tiempo real.
   - Cada vez que el usuario escribe en el campo de texto, el número de caracteres se actualiza y se muestra en pantalla.
</details>
-->
### **Ejercicio 4: Botón para borrar el valor de entrada**
**Problema**: Crea una aplicación donde se borre el texto ingresado en un `OutlinedTextField` al presionar un `OutlinedButton`.

**Ejemplo de diseño:**
<br>
![image](https://github.com/user-attachments/assets/34e92eb5-16de-4423-8729-be10bd21e357)

<!--
<details>
  <summary>Ejemplo de solución</summary>

   ```kotlin
   @Composable
   fun ClearTextFieldExample() {
       var text by remember { mutableStateOf("") }

       Column(
           modifier = Modifier.padding(16.dp),
           horizontalAlignment = Alignment.CenterHorizontally
       ) {
           OutlinedTextField(
               value = text,
               onValueChange = { text = it },
               label = { Text("Ingresa texto") }
           )
           Spacer(modifier = Modifier.height(16.dp))
           OutlinedButton(onClick = { text = "" }) {
               Text("Borrar")
           }
       }
   }
   ```

   **Explicación**:  
   - `OutlinedTextField` guarda el valor ingresado en la variable `text` a través de `onValueChange`.
   - En el `onClick` del botón, `text` se vacía, lo que limpia el contenido del campo de texto.
</details>
-->

---

### **Ejercicio 5: Borrar múltiples `OutlinedTextField` con un solo botón**
**Problema**: Crea una aplicación donde haya dos `OutlinedTextField` y un solo botón para borrar ambos campos.

**Ejemplo de diseño:**
<br>
![image](https://github.com/user-attachments/assets/fe25f9f1-77a9-421a-9e8a-f17f7c59a3e1)


### **Ejercicio 6: Múltiples campos**
**Problema**: Crea dos `OutlinedTextField` donde el usuario pueda ingresar su apellido y edad. Al presionar el botón de "Enviar", debería mostrarse en la pantalla: "[Apellido] tiene [Edad] años".

**Ejemplo de diseño:**
<br>
![image](https://github.com/user-attachments/assets/62c449f6-603e-42cf-b5d0-249cdcdf59de)

### ** Ejemplo con funcionalidad **
**Calculadora de Descuentos**, en este ejemplo de crea un pequeña calculadora que captura el precio de algo(puede ser un producto), el porcentaje de descuento y calcula el total y el ahororo obtenido, haciendo uso de gestión de estado y creando funcionalidad a los eventos clicks de los botones
* Función principal
```kotlin
@Composable
fun DiscountCalculator(modifier: Modifier) {
    //definimos variables de estado
    var precio by remember { mutableStateOf("") }
    var descuento by remember { mutableStateOf("") }
    var total by remember { mutableStateOf(0.0) }
    var ahorro by remember { mutableStateOf(0.0) }
    var error by remember { mutableStateOf<String?>(null) }

    Column(
        modifier = modifier.fillMaxSize().padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            //llamamos la función que pinta el card, reutilizandola para mostrar ambos datos
            ResultCard("Total", total, Modifier.weight(1f))
            ResultCard("Ahorro", ahorro, Modifier.weight(1f))
        }

        //Creamos los OutlinedTextField para capturar precio y descuento
        OutlinedTextField(
            value = precio,
            onValueChange = {precio = it},
            label = {Text("Precio")},
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Decimal),
            singleLine = true,
            isError = error != null,
            modifier = Modifier.fillMaxWidth()
        )

        OutlinedTextField(
            value = descuento,
            onValueChange = {descuento = it},
            label = {Text("Descuento (%)")},
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Decimal),
            singleLine = true,
            isError = error != null,
            modifier = Modifier.fillMaxWidth()
        )
        //Mostramos mensaje en caso de error
        error?.let {
            Text(text = it, color = MaterialTheme.colorScheme.error)
        }
        //Agregamos el boton para hacer los calculos
        Button(
            onClick = {
                //convertimos a Double precio y descuento
                val p = precio.toDoubleOrNull()
                val d = descuento.toDoubleOrNull()
                when {
                    p == null || d == null -> error = "Digite valores numéricos válidos"
                    d !in 0.0..100.0 -> error  = "El descuento debe ser entre 0.0 y 100"
                    else -> {
                        ahorro = p * d / 100
                        total = p - ahorro
                        error = null
                    }
                }
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Calcular")
        }
        //Boton para limpiar datos
        OutlinedButton(
            onClick = {
                precio = ""
                descuento = ""
                total = 0.0
                ahorro = 0.0
                error = null
            },
            colors = ButtonDefaults.outlinedButtonColors(
                contentColor = MaterialTheme.colorScheme.error
            ),
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Limpiar")
        }
    }
}
```
* Función que pinta los cards para mostrar los resultados
```kotlin
@Composable
fun ResultCard(titulo: String, valor: Double, modifier: Modifier = Modifier){
    Card(modifier = modifier) {
        Column(modifier = Modifier.fillMaxWidth().padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(text = titulo, style = MaterialTheme.typography.labelLarge)
            Text(
                text = "\$%.2f".format(valor)
            )
        }
    }
}
```
* llamar la función principal en onCreate
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            DemoStateTheme {
                //llamamos la funcion
               //OutlinedButtonMessage()
                //BasicTextFieldExample()
                //OutlinedButtonContador()

                Scaffold(modifier = Modifier.fillMaxSize()) {innerPadding ->
                    //invocamos la función principal
                    DiscountCalculator(modifier = Modifier.padding(innerPadding))
                }

            }
        }
    }
}
```
