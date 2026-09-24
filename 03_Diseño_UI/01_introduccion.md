# Diseño UI con JetPack Compose

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

## Funciones composables para el diseño de la UI
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
