# Guía 2: Diseño UI con Jetpack Compose — Listas y Campos de Entrada

Segunda guía de apoyo del tema de Diseño UI. Cubre los composables más usados para mostrar listas (`LazyColumn`, `LazyRow`) y capturar texto del usuario (`TextField`, `OutlinedTextField`), además de otros elementos de selección comunes.

## **1. Objetivos**

- Mostrar listas de datos de forma eficiente con `LazyColumn` y `LazyRow`.
- Capturar texto del usuario con `TextField` y `OutlinedTextField`.
- Identificar otros composables de entrada frecuentes: `Checkbox`, `Switch`, `RadioButton`.

---

## **2. `LazyColumn` — listas verticales**

Ya se vio en el ejemplo Column y Row: `LazyColumn` dibuja únicamente los elementos visibles en pantalla, lo que la hace eficiente para listas largas (a diferencia de un `Column` con muchos hijos, que dibuja todo de una vez).

```kotlin
@Composable
fun ListaDeNombres(nombres: List<String>) {
    LazyColumn(
        modifier = Modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(nombres) { nombre ->
            Text(text = nombre, style = MaterialTheme.typography.bodyLarge)
        }
    }
}
```

- `contentPadding`: a diferencia de `Modifier.padding`, este sí respeta el área de scroll (el padding no desaparece al hacer scroll).
- `verticalArrangement = Arrangement.spacedBy(8.dp)`: separa cada elemento de la lista por 8dp, sin necesidad de agregarle `Modifier.padding` a cada uno.
- `items(nombres) { nombre -> ... }`: recorre la lista y dibuja el contenido una vez por cada elemento.

### Variante: `itemsIndexed`

Cuando se necesita también la posición del elemento en la lista:

```kotlin
LazyColumn {
    itemsIndexed(nombres) { index, nombre ->
        Text(text = "${index + 1}. $nombre")
    }
}
```

---

## **3. `LazyRow` — listas horizontales**

Igual que `LazyColumn`, pero organiza los elementos en una fila horizontal con scroll. Útil para carruseles de categorías, imágenes o "chips".

```kotlin
@Composable
fun CarruselCategorias(categorias: List<String>) {
    LazyRow(
        contentPadding = PaddingValues(horizontal = 16.dp),
        horizontalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        items(categorias) { categoria ->
            AssistChip(
                onClick = { },
                label = { Text(categoria) }
            )
        }
    }
}
```

- `AssistChip` es un composable de Material 3 para mostrar una etiqueta accionable (por ejemplo, un filtro).
- La lógica es idéntica a `LazyColumn`: la única diferencia real es la dirección del scroll.

---

## **4. `TextField` — campo de texto básico**

`TextField` es un campo de texto editable. A diferencia de `Text`, necesita **estado**: un valor que se actualiza cada vez que el usuario escribe.

```kotlin
@Composable
fun CampoNombre() {
    var nombre by remember { mutableStateOf("") }

    TextField(
        value = nombre,
        onValueChange = { nuevoValor -> nombre = nuevoValor },
        label = { Text("Nombre") },
        modifier = Modifier.fillMaxWidth()
    )
}
```

- `value`: el texto que se muestra actualmente en el campo.
- `onValueChange`: se ejecuta cada vez que el usuario escribe algo nuevo — es responsabilidad del desarrollador actualizar el estado (`nombre = nuevoValor`); si no se hace, el campo se ve "congelado" y no permite escribir.
- `var nombre by remember { mutableStateOf("") }`: aquí se aplica directamente a un campo de texto.

---

## **5. `OutlinedTextField` — campo de texto con borde**

Funciona exactamente igual que `TextField` en cuanto a parámetros y lógica; la única diferencia es visual: dibuja un borde alrededor del campo en vez de un fondo relleno.

```kotlin
@Composable
fun CampoCorreo() {
    var correo by remember { mutableStateOf("") }

    OutlinedTextField(
        value = correo,
        onValueChange = { correo = it },
        label = { Text("Correo electrónico") },
        keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
        singleLine = true,
        modifier = Modifier.fillMaxWidth()
    )
}
```

- `{ correo = it }`: forma abreviada de `{ nuevoValor -> correo = nuevoValor }` — `it` es el nombre automático que Kotlin da al único parámetro de una lambda cuando no se nombra explícitamente.
- `keyboardOptions`: ajusta el tipo de teclado que aparece (aquí, uno optimizado para correos).
- `singleLine = true`: evita que el campo crezca a múltiples líneas al escribir texto largo.

**¿Cuál usar?** Es una decisión puramente visual/de diseño: `OutlinedTextField` suele preferirse en formularios sobre fondo claro porque se distingue mejor del fondo.

---

## **6. Otros elementos de selección comunes**

### `Checkbox`

```kotlin
var aceptaTerminos by remember { mutableStateOf(false) }

Row(verticalAlignment = Alignment.CenterVertically) {
    Checkbox(checked = aceptaTerminos, onCheckedChange = { aceptaTerminos = it })
    Text("Acepto los términos y condiciones")
}
```

### `Switch`

```kotlin
var notificacionesActivas by remember { mutableStateOf(true) }

Row(verticalAlignment = Alignment.CenterVertically) {
    Text("Notificaciones")
    Spacer(modifier = Modifier.width(8.dp))
    Switch(checked = notificacionesActivas, onCheckedChange = { notificacionesActivas = it })
}
```

### `RadioButton`

```kotlin
var opcionSeleccionada by remember { mutableStateOf("Efectivo") }
val opciones = listOf("Efectivo", "Tarjeta")

Column {
    opciones.forEach { opcion ->
        Row(verticalAlignment = Alignment.CenterVertically) {
            RadioButton(
                selected = (opcion == opcionSeleccionada),
                onClick = { opcionSeleccionada = opcion }
            )
            Text(opcion)
        }
    }
}
```

Los tres siguen el mismo patrón que `TextField`: un valor de estado (`checked`, `selected`) y una función que actualiza ese estado cuando el usuario interactúa (`onCheckedChange`, `onClick`).

---

## **7. Ejercicio propuesto**

**Pantalla: "Agregar contacto"**

Construir una pantalla que permita registrar contactos y verlos en una lista, usando únicamente los elementos vistos en esta guía (y las anteriores). Requisitos:

1. Un formulario en la parte superior con:
   - Un `OutlinedTextField` para el **nombre**.
   - Un `OutlinedTextField` para el **teléfono** (usar `KeyboardType.Phone`).
   - Un `Checkbox` para marcar el contacto como **"Favorito"**.
2. Un botón **"Agregar"** que:
   - Tome los valores actuales del formulario.
   - Los agregue a una lista de contactos (puede ser una `MutableList` en un `remember`, todavía sin `ViewModel` ni Room — eso se verá más adelante).
   - Limpie el formulario después de agregar (los campos vuelven a quedar vacíos).
3. Debajo del formulario, un `LazyColumn` que muestre todos los contactos agregados, cada uno en una `Card` con:
   - El nombre.
   - El teléfono.
   - Una estrella o texto "★ Favorito" visible **solo** si el contacto se marcó como favorito.
4. **Reto opcional:** agregar un `LazyRow` con `AssistChip`s para filtrar la lista por "Todos" / "Favoritos".

**Pistas para el estudiante (sin dar la solución completa):**

- El estado de la lista de contactos necesita `remember { mutableStateListOf<Contacto>() }` en vez de `mutableStateOf(listOf(...))`, para poder agregar elementos sin reemplazar toda la lista.
- Definir primero `data class Contacto(val nombre: String, val telefono: String, val favorito: Boolean)`, como se hizo con `Tarea` en la guía anterior.
- El botón "Agregar" debe validar que el campo de nombre no esté vacío antes de agregar (se puede usar un simple `if`).
