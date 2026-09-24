# Diseño UI con JetPack Compose
## Método onCreate de MainActivity
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            SampleTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    Box(
                        modifier = Modifier
                            .fillMaxSize()
                            .padding(innerPadding),
                        contentAlignment = Alignment.Center
                    ) {
                        UserFrofile(
                            name = "Carlos Zelaya",
                            role = "Developer",
                            location = "Chalatenango Sur",
                            email = "carlos.zelaya@empresa.com",
                            onSiguemeClick = {
                                // Acción para el botón Sigueme
                            },
                            onContactClick = {
                                {/* Acción para el boton Contact */}
                            }
                        )
                    }
                }
            }
        }
    }
}
```

## Función composable UserProfile
```kotlin
@Composable
fun UserFrofile(
    name: String,
    role: String,
    location: String,
    email: String,
    modifier: Modifier = Modifier,
    onSiguemeClick: () -> Unit = {},
    onContactClick: () -> Unit = {}
) {
    // Contexto de Android necesario para invocar el Toast
    val context = LocalContext.current

    // Definiendo colores a utilizar
    val darkHeaderColor = Color(0xFF2C3946)      // Encabezado superior
    val lightBodyColor = Color(0xFFEDE9F2)       // Contenedor inferior lila suave
    val tealAccentColor = Color(0xFF00D28E)      // Verde turquesa de los iconos de redes sociales
    val outlineBorderColor = Color(0xFF6B7280)   // Borde sutil del botón Follow

    ElevatedCard(
        modifier = modifier
            .fillMaxWidth()
            .padding(horizontal = 20.dp),
        shape = RoundedCornerShape(24.dp),
        colors = CardDefaults.elevatedCardColors(
            containerColor = MaterialTheme.colorScheme.surface
        ),
        elevation = CardDefaults.elevatedCardElevation(
            defaultElevation = 6.dp
        )
    ) {
        // Definimos un contenedor Column para distribuir verticalmente el contenido
        Column(
            modifier = Modifier.fillMaxWidth()
        ) {
            // SECCIÓN 1: ENCABEZADO OSCURO (Avatar + Textos)

            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .background(darkHeaderColor)
                    .padding(horizontal = 20.dp, vertical = 20.dp),
                verticalAlignment = Alignment.CenterVertically,
                horizontalArrangement = Arrangement.spacedBy(16.dp)
            ) {
                // Box con el Avatar y su estado activo
                Box(
                    modifier = Modifier.size(72.dp),
                    contentAlignment = Alignment.Center
                ) {
                    // Colocamos un Avatar tipo squircle
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
                                contentDescription = "Avatar del usuario",
                                modifier = Modifier.size(44.dp)
                            )
                        }
                    }

                    // Otro surface: punto indicador de estado activo
                    Surface(
                        modifier = Modifier
                            .size(16.dp)
                            .align(Alignment.BottomEnd),
                        shape = CircleShape,
                        color = MaterialTheme.colorScheme.primary,
                        border = BorderStroke(
                            width = 2.5.dp,
                            color = darkHeaderColor
                        )
                    ) {}
                }

                // Datos de identificación textual (Nombre y Cargo)
                Column(
                    modifier = Modifier.weight(1f),
                    verticalArrangement = Arrangement.spacedBy(2.dp)
                ) {
                    Text(
                        text = name,
                        color = Color.White,
                        fontSize = 20.sp,
                        fontWeight = FontWeight.Bold
                    )
                    Text(
                        text = role,
                        color = Color(0xFF94A3B8), // Gris suave
                        fontSize = 14.sp,
                        fontWeight = FontWeight.Normal
                    )
                }
            }

            // SECCIÓN 2: CUERPO CLARO (Redes Sociales + Acciones)

            Column(
                modifier = Modifier
                    .fillMaxWidth()
                    .background(lightBodyColor)
                    .padding(horizontal = 20.dp, vertical = 22.dp)
            ) {
                // Fila con los 4 botones sociales circulares verde menta
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.SpaceBetween,
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    // Botón 1: Twitter / Compartir[cite: 1]
                    Box(
                        modifier = Modifier
                            .size(48.dp)
                            .clip(CircleShape)
                            .background(tealAccentColor)
                            .clickable {
                                Toast.makeText(context, "Abriendo X de $name", Toast.LENGTH_SHORT).show()
                            },
                        contentAlignment = Alignment.Center
                    ) {
                        Text(
                            text = "𝕏", // Carácter unicode oficial de la marca
                            color = Color.White,
                            fontSize = 20.sp,
                            fontWeight = FontWeight.ExtraBold

                        )
                    }

                    // Botón 2: GitHub / Código
                    Box(
                        modifier = Modifier
                            .size(48.dp)
                            .clip(CircleShape)
                            .background(tealAccentColor)
                            .clickable { },
                        contentAlignment = Alignment.Center
                    ) {
                        Icon(
                            imageVector = Icons.Default.Code,
                            contentDescription = "GitHub",
                            tint = Color.White,
                            modifier = Modifier.size(22.dp)
                        )
                    }

                    // Botón 3: LinkedIn / Red profesional
                    Box(
                        modifier = Modifier
                            .size(48.dp)
                            .clip(CircleShape)
                            .background(tealAccentColor)
                            .clickable { },
                        contentAlignment = Alignment.Center
                    ) {
                        Icon(
                            imageVector = Icons.Default.Public,
                            contentDescription = "LinkedIn",
                            tint = Color.White,
                            modifier = Modifier.size(22.dp)
                        )
                    }

                    // Botón 4: Blog / Web
                    Box(
                        modifier = Modifier
                            .size(48.dp)
                            .clip(CircleShape)
                            .background(tealAccentColor)
                            .clickable { },
                        contentAlignment = Alignment.Center
                    ) {
                        Icon(
                            imageVector = Icons.Default.Language,
                            contentDescription = "Sitio Web",
                            tint = Color.White,
                            modifier = Modifier.size(22.dp)
                        )
                    }
                }

                Spacer(modifier = Modifier.height(20.dp))

                // Fila de botones de acción: Sigueme y Contact
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.spacedBy(12.dp)
                ) {
                    // Botón contorneado Follow[cite: 1]
                    OutlinedButton(
                        onClick = onSiguemeClick,
                        modifier = Modifier
                            .weight(1f)
                            .height(46.dp),
                        shape = RoundedCornerShape(12.dp),
                        border = BorderStroke(1.5.dp, outlineBorderColor),
                        colors = ButtonDefaults.outlinedButtonColors(
                            contentColor = darkHeaderColor
                        )
                    ) {
                        Text(
                            text = "Sígueme",
                            fontWeight = FontWeight.SemiBold,
                            fontSize = 15.sp
                        )
                    }

                    // Botón relleno Contact
                    Button(
                        onClick = onContactClick,
                        modifier = Modifier
                            .weight(1f)
                            .height(46.dp),
                        shape = RoundedCornerShape(12.dp),
                        colors = ButtonDefaults.buttonColors(
                            containerColor = darkHeaderColor,
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
            UserFrofile(
                name = "Carlos Zelaya",
                role = "Developer",
                location = "Chalatenango Sur",
                email = "carlos.zelaya@empresa.com"
            )
        }
    }
}
```
