# Diseño UI con Jetpack Compose

Introducción a Jetpack Compose y los elementos básicos de la UI

## **1. Objetivos del tema**

- Comprender los conceptos básicos de Jetpack Compose.
- Aprender a utilizar la anotación `@Composable`.
- Aprender cómo comprobar el diseño en Android Studio.

---

## **2. Introducción a Jetpack Compose**

### **¿Qué es Jetpack Compose?**

Jetpack Compose es el último conjunto de herramientas de UI para Android, diseñado para hacer que la creación de interfaces sea más simple e intuitiva. A diferencia de los diseños basados en XML, Compose permite describir la UI directamente en código Kotlin, adoptando un enfoque de programación reactiva.

### **Ventajas de Jetpack Compose**

- **Código simple**: Todo el diseño se maneja dentro de código Kotlin, lo que mejora la legibilidad.
- **UI reactiva**: La UI se actualiza automáticamente cuando cambia el estado.
- **Reutilización**: Puedes crear componentes UI reutilizables a través de funciones `@Composable`.

---

## **3. Funciones componibles**

### **Anotación `@Composable`**

Todos los componentes UI en Jetpack Compose se definen como funciones anotadas con `@Composable`. Esto permite que los componentes se rendericen en la pantalla. Se llama `Funciones componibles`.

### **Diseño de UI intuitivo y flexible**

Jetpack Compose es como construir con piezas de Lego. Cada componente de la interfaz de usuario, como `Button` o `Text`, es como una pieza de Lego que puede combinar de diferentes formas para crear el diseño que quieras. Al igual que cuando juega con Lego, puede armar su interfaz visualmente sin necesidad de herramientas complicadas, solo con código.

#### **Ejemplo: Mostrar un texto simple**

```kotlin
@Composable
fun Saludo() {
    Text(text = "¡Hola, Jetpack Compose!")
}
```

#### **Ejemplo: Mostrar un Button simple**

```kotlin
@Composable
fun BotonSimple() {
    Button(onClick = { /* Acción cuando se pulsa el botón */ }) {
        Text("Haz clic aquí")
    }
}
```

**Antes del siguiente ejemplo:** la pantalla completa de la calculadora introduce varios elementos que no se han visto todavía — conviene señalarlos en clase antes de leer el código completo:

- `Row`: organiza elementos horizontalmente (igual que `Column` los organiza verticalmente).
- `Card`: contenedor con apariencia de tarjeta.
- `Modifier.weight(1f)`: dentro de un `Row`, reparte el espacio disponible entre varios elementos en partes iguales.
- `Color` y `fontSize = ...sp`: color y tamaño de texto explícitos.
- `OutlinedTextField` + `KeyboardOptions`: campo de texto editable, aquí configurado para mostrar teclado numérico.

#### **Ejemplo: Mostrar una pantalla de la aplicación**

```kotlin
package com.example.ejemploscomponentes

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.Card
import androidx.compose.material3.OutlinedButton
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.ejemploscomponentes.ui.theme.EjemplosComponentesTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            EjemplosComponentesTheme {
            }
        }
    }
}

@Composable
fun CalculatorApp() {
    Column(modifier= Modifier
        .padding(10.dp)
        .fillMaxSize(),
        //verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        // Tarjetas con el importe total y el descuento
        Row(modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceEvenly
        ) {
            // Tarjeta con el importe total
            Card(
                modifier = Modifier.padding(8.dp).weight(1f)
            ) {
                Column(
                    verticalArrangement = Arrangement.Center,
                    horizontalAlignment = Alignment.CenterHorizontally,
                    modifier = Modifier.padding(16.dp)
                ) {
                    Text(text = "Total", color = Color.Black, fontSize = 20.sp)
                    Text(text = "$0.0", color = Color.Black, fontSize = 20.sp)
                }
            }
            // Tarjeta con el descuento
            Card(
                modifier = Modifier.padding(8.dp).weight(1f)
            ) {
                Column(
                    verticalArrangement = Arrangement.Center,
                    horizontalAlignment = Alignment.CenterHorizontally,
                    modifier = Modifier.padding(16.dp)
                ) {
                    Text(text = "Descuento", color = Color.Black, fontSize = 20.sp)
                    Text(text = "$0.0", color = Color.Black, fontSize = 20.sp)
                }
            }
        }
        // TextField para introducir el precio
        OutlinedTextField(
            value = "",
            onValueChange = {},
            label = { Text(text = "Precio")},
            //mostrando el teclado numerico
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number),
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 32.dp)
        )
        // TextField para introducir el porcentaje de descuento
        OutlinedTextField(
            value = "",
            onValueChange = {},
            label = { Text(text = "Descuento %")},
            //mostrando el teclado numerico
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number),
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 32.dp)
        )
        // Button para generar descuento
        OutlinedButton(
            onClick = { },
            modifier = Modifier.fillMaxWidth().padding(horizontal = 30.dp)
        ) {
            Text(text = "Generar Descuento")
        }
        // Button para Limpiar
        OutlinedButton(
            onClick = {},
            colors = ButtonDefaults.outlinedButtonColors(contentColor = Color.Red),
            modifier = Modifier.fillMaxWidth().padding(horizontal = 30.dp)
        ) {
            Text(text = "Limpiar")
        }
    }
}

@Preview(showBackground = true)
@Composable
fun CalculatorAppPreview() {
    EjemplosComponentesTheme {
        CalculatorApp()
    }
}
```


## **4. ¿Qué es Modifier en Jetpack Compose?**

`Modifier` en Jetpack Compose es una herramienta que te permite personalizar y ajustar la apariencia y el comportamiento de los elementos de la interfaz de usuario (**Composables**).

Es un mecanismo flexible que se usa para tareas como:

- Cambiar el tamaño o posición.
- Agregar márgenes o relleno.
- Aplicar colores de fondo.
- Detectar eventos como clics.

---

### **Sintaxis básica**

Se aplica un `Modifier` directamente a un Composable:

```kotlin
Text(
    text = "¡Hola, Modifier!",
    modifier = Modifier
        .padding(16.dp) // Relleno
        .background(Color.Cyan) // Color de fondo
)
```

### **Ejemplos comunes de Modifier**

#### 1. **Relativo al diseño (Layout)**

- **padding**: Agrega espacio interno alrededor del contenido.

  ```kotlin
  Modifier.padding(8.dp)
  ```
- **size**: Define un tamaño fijo para el Composable.

  ```kotlin
  Modifier.size(100.dp)
  ```
- **fillMaxSize / fillMaxWidth / fillMaxHeight**: Expande el Composable para ocupar todo el espacio disponible.

  ```kotlin
  Modifier.fillMaxSize()
  ```

#### 2. **Decoración**

- **background**: Define un color o una imagen de fondo.

  ```kotlin
  Modifier.background(Color.Red)
  ```
- **border**: Agrega un borde al Composable.

  ```kotlin
  Modifier.border(2.dp, Color.Black)
  ```

---

### **¿A qué equivale `Modifier` en JavaScript?**

En el contexto de JavaScript, `Modificador` es una equivalente de las **propiedades CSS**. `Modifier` se utiliza para ajustar el **estilo y el diseño** de los elementos de la interfaz, algo muy similar a lo que hacemos con CSS.

#### Jetpack Compose (`Modifier`)

```kotlin
Box(
    modifier = Modifier
        .size(100.dp)
        .padding(16.dp)
        .background(Color.Gray)
)
```

#### HTML + CSS

```html
<div style="
    width: 100px;
    height: 100px;
    padding: 16px;
    background-color: gray;
"></div>
```

En este caso, las funciones de `Modifier` como `size`, `padding` o `background` se asemejan a las propiedades CSS para definir tamaño, márgenes internos y color de fondo.

---

## **5. Segundo proyecto de práctica: tarjeta de perfil (`Sample`)**

Este segundo ejemplo es un proyecto NUEVO y separado del anterior (`EjemplosComponentes`), pensado para practicar composables más avanzados: tarjetas elevadas, iconos, retroalimentación con `Toast`, y cómo dividir un componente grande en varias funciones más pequeñas y reutilizables.

**Antes de copiar el código — dependencia a agregar:** más adelante se usan los iconos `Icons.Default.Code`, `Icons.Default.Public` y `Icons.Default.Language`. Estos NO vienen incluidos en el set básico de iconos (`material-icons-core`, que se agrega automáticamente con Compose). Hace falta agregar la librería extendida en `build.gradle.kts` (módulo `app`):

```kotlin
dependencies {
    implementation("androidx.compose.material:material-icons-extended")
}
```

`Icons.Default.AccountCircle` y `Icons.Default.Email`, en cambio, sí forman parte del set básico y no requieren esta dependencia.

## Definición de variables inmutables para colores de la vista, colocar arriba de **class MainAcitvity**

```kotlin
// variables inmutables con colores a utilizar a nivel de Activity
private val DarkHeaderColor = Color(0xFF2C3946)
private val LightBodyColor = Color(0xFFEDE9F2)
private val TealAccentColor = Color(0xFF00D28E)
private val OutlineBorderColor = Color(0xFF6B7280)
```

## Método onCreate de MainActivity

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            SampleTheme {
                //obtenemos el contexto para mostrar mensaje tipo Toast
                val context = LocalContext.current
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    Box(
                        modifier = Modifier
                            .fillMaxSize()
                            .padding(innerPadding),
                        contentAlignment = Alignment.Center
                    ) {
                        //invocar la función principal que dibuja la vista y pasar parámetros
                        UserProfile(
                            name = "Carlos Zelaya",
                            role = "Developer",
                            email = "carlos.zelaya@empresa.com",
                            onSiguemeClick = {
                                Toast.makeText(context, "Siguiendo a Carlos Zelaya", Toast.LENGTH_SHORT).show()
                            },
                            onContactClick = {
                                Toast.makeText(context, "Contactando a carlos.zelaya@empresa.com", Toast.LENGTH_SHORT).show()
                            }
                        )
                    }
                }
            }
        }
    }
}
```

### Funciones composables para el diseño de la UI

Antes del código completo, algunos elementos que aparecen aquí por primera vez:

- `ElevatedCard` / `CardDefaults`: variante de `Card` con más control sobre su elevación (sombra) y colores.
- `RoundedCornerShape`: define bordes redondeados con un radio específico (a diferencia del `CircleShape`, que redondea completamente).
- `Icon` + `ImageVector`: muestra un ícono de Material Icons.
- `BorderStroke`: dibuja un borde alrededor de un `Surface` o `Box`.
- `tonalElevation`: sombra/tono adicional propio de Material 3, distinto de la elevación tradicional.
- `Toast` + `LocalContext.current`: forma de mostrar un mensaje corto al usuario; `LocalContext` es cómo Compose accede al `Context` de Android dentro de una función `@Composable`.
- `Arrangement.spacedBy(...)`: separa a los hijos de un `Row`/`Column` con un espacio fijo entre cada uno (alternativa a colocar un `Spacer` manualmente entre cada elemento).

```kotlin
@Composable
fun UserProfile(
    name: String,
    role: String,
    email: String,
    modifier: Modifier = Modifier,
    onSiguemeClick: () -> Unit = {},
    onContactClick: () -> Unit = {}
) {
    ElevatedCard(
        modifier = modifier
            .fillMaxWidth()
            .padding(horizontal = 20.dp),
        shape = RoundedCornerShape(24.dp),
        colors = CardDefaults.elevatedCardColors(
            containerColor = MaterialTheme.colorScheme.surface
        ),
        elevation = CardDefaults.elevatedCardElevation(defaultElevation = 6.dp)
    ) {
        Column(modifier = Modifier.fillMaxWidth()) {
            ProfileHeader(name = name, role = role, email = email)

            Column(
                modifier = Modifier
                    .fillMaxWidth()
                    .background(LightBodyColor)
                    .padding(horizontal = 20.dp, vertical = 22.dp)
            ) {
                SocialMediaBar(name = name)

                Spacer(modifier = Modifier.height(20.dp))

                ActionButtons(
                    onSiguemeClick = onSiguemeClick,
                    onContactClick = onContactClick
                )
            }
        }
    }
}


// Sección superior oscura: Avatar con status badge, nombre, cargo y correo.
@Composable
fun ProfileHeader(
    name: String,
    role: String,
    email: String,
    modifier: Modifier = Modifier
) {
    Row(
        modifier = modifier
            .fillMaxWidth()
            .background(DarkHeaderColor)
            .padding(horizontal = 20.dp, vertical = 20.dp),
        verticalAlignment = Alignment.CenterVertically,
        horizontalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        // Avatar con borde squircle e indicador de estado
        Box(
            modifier = Modifier.size(72.dp),
            contentAlignment = Alignment.Center
        ) {
            Surface(
                modifier = Modifier
                    .size(68.dp)
                    .border(
                        width = 1.5.dp,
                        color = MaterialTheme.colorScheme.primary.copy(alpha = 0.35f),
                        shape = RoundedCornerShape(20.dp)
                    ),
                shape = RoundedCornerShape(20.dp),
                color = MaterialTheme.colorScheme.primaryContainer,
                contentColor = MaterialTheme.colorScheme.onPrimaryContainer,
                tonalElevation = 4.dp
            ) {
                Box(contentAlignment = Alignment.Center) {
                    Icon(
                        imageVector = Icons.Default.AccountCircle,
                        contentDescription = "Avatar de $name",
                        modifier = Modifier.size(44.dp)
                    )
                }
            }

            Surface(
                modifier = Modifier
                    .size(16.dp)
                    .align(Alignment.BottomEnd),
                shape = CircleShape,
                color = MaterialTheme.colorScheme.primary,
                border = BorderStroke(width = 2.5.dp, color = DarkHeaderColor)
            ) {}
        }

        // Información textual: Nombre, Cargo y Correo
        Column(
            modifier = Modifier.weight(1f),
            verticalArrangement = Arrangement.spacedBy(3.dp)
        ) {
            Text(
                text = name,
                color = Color.White,
                fontSize = 20.sp,
                fontWeight = FontWeight.Bold
            )
            Text(
                text = role,
                color = Color(0xFF94A3B8),
                fontSize = 14.sp,
                fontWeight = FontWeight.Normal
            )

            // Fila con el correo electrónico y su icono
            Row(
                verticalAlignment = Alignment.CenterVertically,
                horizontalArrangement = Arrangement.spacedBy(4.dp)
            ) {
                Icon(
                    imageVector = Icons.Default.Email,
                    contentDescription = "Email",
                    tint = TealAccentColor,
                    modifier = Modifier.size(14.dp)
                )
                Text(
                    text = email,
                    color = Color(0xFFCBD5E1),
                    fontSize = 12.sp,
                    fontWeight = FontWeight.Normal
                )
            }
        }
    }
}

// Barra horizontal con los 4 accesos directos a redes sociales.
@Composable
fun SocialMediaBar(
    name: String,
    modifier: Modifier = Modifier
) {
    val context = LocalContext.current

    Row(
        modifier = modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        SocialCircleButton(
            onClick = {
                Toast.makeText(context, "Abriendo X de $name", Toast.LENGTH_SHORT).show()
            }
        ) {
            Text(
                text = "𝕏",
                color = Color.White,
                fontSize = 20.sp,
                fontWeight = FontWeight.ExtraBold
            )
        }

        SocialCircleButton(
            icon = Icons.Default.Code,
            contentDescription = "GitHub",
            onClick = {
                Toast.makeText(context, "Abriendo GitHub de $name", Toast.LENGTH_SHORT).show()
            }
        )

        SocialCircleButton(
            icon = Icons.Default.Public,
            contentDescription = "LinkedIn",
            onClick = {
                Toast.makeText(context, "Abriendo LinkedIn de $name", Toast.LENGTH_SHORT).show()
            }
        )

        SocialCircleButton(
            icon = Icons.Default.Language,
            contentDescription = "Sitio Web",
            onClick = {
                Toast.makeText(context, "Visitando sitio web", Toast.LENGTH_SHORT).show()
            }
        )
    }
}


// Componente base circular para evitar duplicar código en cada botón de red social.

@Composable
fun SocialCircleButton(
    modifier: Modifier = Modifier,
    icon: ImageVector? = null,
    contentDescription: String? = null,
    onClick: () -> Unit = {},
    customContent: (@Composable () -> Unit)? = null
) {
    Box(
        modifier = modifier
            .size(48.dp)
            .clip(CircleShape)
            .background(TealAccentColor)
            .clickable { onClick() },
        contentAlignment = Alignment.Center
    ) {
        if (customContent != null) {
            customContent()
        } else if (icon != null) {
            Icon(
                imageVector = icon,
                contentDescription = contentDescription,
                tint = Color.White,
                modifier = Modifier.size(22.dp)
            )
        }
    }
}

// Fila inferior con las dos acciones principales: Sígueme y Contact.

@Composable
fun ActionButtons(
    onSiguemeClick: () -> Unit,
    onContactClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Row(
        modifier = modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        OutlinedButton(
            onClick = onSiguemeClick,
            modifier = Modifier
                .weight(1f)
                .height(46.dp),
            shape = RoundedCornerShape(12.dp),
            border = BorderStroke(1.5.dp, OutlineBorderColor),
            colors = ButtonDefaults.outlinedButtonColors(
                contentColor = DarkHeaderColor
            )
        ) {
            Text(
                text = "Sígueme",
                fontWeight = FontWeight.SemiBold,
                fontSize = 15.sp
            )
        }

        Button(
            onClick = onContactClick,
            modifier = Modifier
                .weight(1f)
                .height(46.dp),
            shape = RoundedCornerShape(12.dp),
            colors = ButtonDefaults.buttonColors(
                containerColor = DarkHeaderColor,
                contentColor = Color.White
            )
        ) {
            Text(
                text = "Contact",
                fontWeight = FontWeight.SemiBold,
                fontSize = 15.sp
            )
        }
    }
}
```

## Función para crear la vista previa

```kotlin
@Preview(showBackground = true)
@Composable
fun UserProfilePreview() {
    SampleTheme {
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(16.dp),
            contentAlignment = Alignment.Center
        ) {
            UserProfile(
                name = "Carlos Zelaya",
                role = "Developer",
                email = "carlos.zelaya@empresa.com"
            )
        }
    }
}
```
