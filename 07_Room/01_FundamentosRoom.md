# 7\_1 — Persistencia de datos con Room: Agenda de Contactos

Guía para persistir datos localmente con **Room**, usando como ejemplo una agenda de contactos (nombre, teléfono, correo y una foto tomada con la cámara o elegida de la galería).

## **1. Objetivos**

- Comprender qué es Room y cuándo conviene usarlo.
- Crear una entidad, un DAO y una base de datos con Room.
- Conectar Room con la UI a través de un `ViewModel`.
- Capturar una foto con la cámara o seleccionarla de la galería y guardar su referencia en la base de datos.

---

## **2. ¿Qué es Room y por qué usarlo?**

Room es la librería oficial de Android para trabajar con **SQLite** de forma más segura y sencilla: en vez de escribir SQL a mano y manejar cursores, se definen clases Kotlin y Room genera el código necesario en tiempo de compilación.

Se usa Room cuando los datos son **estructurados y se repiten** (una lista de contactos, tareas, productos, etc.). 

---

## **3. Dependencias necesarias**

Cuando se crea el proyecto usa **Gradle Version Catalogs** (`libs.versions.toml`), así que las versiones no van directo en `build.gradle.kts` sino en ese archivo. Hay que agregar tres cosas: el plugin **KSP** (procesador de anotaciones de Room), las librerías de **Room**, y **Coil** (para mostrar la foto del contacto).

**1. En `gradle/libs.versions.toml`, agregar a `[versions]`:**

```toml
ksp = "2.2.10-2.0.2"   # debe coincidir con su versión de Kotlin (aquí 2.2.10)
room = "2.8.5"
coil = "3.0.4"
```

**Agregar a `[plugins]`:**

```toml
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

**Agregar a `[libraries]`:**

```toml
androidx-room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
androidx-room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
androidx-room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
coil-compose = { group = "io.coil-kt.coil3", name = "coil-compose", version.ref = "coil" }
```

**2. En el `build.gradle.kts` de la RAÍZ del proyecto**, agregar el plugin KSP junto a los que ya tiene (con `apply false`):

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.ksp) apply false
}
```

**3. En el `build.gradle.kts` del módulo `app`:**

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.ksp)
}

dependencies {
    implementation(libs.androidx.room.runtime)
    implementation(libs.androidx.room.ktx)
    ksp(libs.androidx.room.compiler)

    implementation(libs.coil.compose)
}
```

Note que los guiones del TOML (`androidx-room-runtime`) se convierten en puntos al usarlos (`libs.androidx.room.runtime`) — eso lo genera Gradle automáticamente, no hay que escribirlo distinto a como está en el archivo.

---

