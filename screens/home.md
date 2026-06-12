# HomeScreen

## Visión General

Pantalla principal que sirve como menú de navegación hacia todas las funcionalidades de la aplicación. Presenta un grid de opciones con iconos y textos descriptivos.

**Archivo**: `presentation/ui/screens/HomeScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  Password Manager        [Perfil]       │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────┐   ┌─────────────┐      │
│  │   🔐        │   │   📋        │      │
│  │ Contraseñas│   │ Categorías  │      │
│  └─────────────┘   └─────────────┘      │
│                                         │
│  ┌─────────────┐   ┌─────────────┐      │
│  │   🎲        │   │   📊        │      │
│  │ Generador  │   │ Estadísticas│      │
│  └─────────────┘   └─────────────┘      │
│                                         │
│  ┌─────────────┐   ┌─────────────┐      │
│  │   🔍        │   │   💾        │      │
│  │  Auditoría │   │   Backup    │      │
│  └─────────────┘   └─────────────┘      │
│                                         │
│  ┌─────────────┐                        │
│  │   ⚙️        │                        │
│  │ Configuración│                       │
│  └─────────────┘                        │
│                                         │
├─────────────────────────────────────────┤
│  [Cerrar Sesión]                        │
└─────────────────────────────────────────┘
```

## Elementos del Grid

| Opción | Icono | Ruta de Navegación | Descripción |
|--------|-------|-------------------|-------------|
| Contraseñas | 🔐 / `FaKey` | `/password_list` | Lista de contraseñas guardadas |
| Categorías | 📋 / `FaFolder` | `/category_management` | Gestión de categorías |
| Generador | 🎲 / `FaDice` | `/password_generator` | Generador de contraseñas |
| Estadísticas | 📊 / `FaChartBar` | `/statistics` | Métricas de seguridad |
| Auditoría | 🔍 / `FaSearch` | `/audit` | Detección de passwords débiles |
| Backup | 💾 / `FaDatabase` | `/backup` | Exportar/importar datos |
| Configuración | ⚙️ / `FaCog` | `/settings` | Ajustes de la app |

## Estado de la UI

```kotlin
data class HomeState(
    val userName: String = "",
    val lastAccess: Long = System.currentTimeMillis(),
    val isLoading: Boolean = false,
    val error: String? = null
)
```

## Código del Componente

```kotlin
@Composable
fun HomeScreen(
    viewModel: AuthViewModel = koinViewModel(),
    onNavigateToPasswords: () -> Unit,
    onNavigateToCategories: () -> Unit,
    onNavigateToGenerator: () -> Unit,
    onNavigateToStatistics: () -> Unit,
    onNavigateToAudit: () -> Unit,
    onNavigateToBackup: () -> Unit,
    onNavigateToSettings: () -> Unit,
    onLogout: () -> Unit,
    modifier: Modifier = Modifier
) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        // Header
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text(
                text = "Password Manager",
                style = MaterialTheme.typography.headlineMedium,
                fontWeight = FontWeight.Bold
            )
            
            IconButton(onClick = { /* Perfil */ }) {
                Icon(
                    imageVector = Icons.Default.Person,
                    contentDescription = "Perfil"
                )
            }
        }
        
        Spacer(modifier = Modifier.height(24.dp))
        
        // Grid de opciones
        val options = listOf(
            HomeOption(
                icon = Icons.Default.Lock,
                title = "Contraseñas",
                onClick = onNavigateToPasswords
            ),
            HomeOption(
                icon = Icons.Default.Folder,
                title = "Categorías",
                onClick = onNavigateToCategories
            ),
            HomeOption(
                icon = Icons.Default.Casino,
                title = "Generador",
                onClick = onNavigateToGenerator
            ),
            HomeOption(
                icon = Icons.Default.BarChart,
                title = "Estadísticas",
                onClick = onNavigateToStatistics
            ),
            HomeOption(
                icon = Icons.Default.Search,
                title = "Auditoría",
                onClick = onNavigateToAudit
            ),
            HomeOption(
                icon = Icons.Default.Storage,
                title = "Backup",
                onClick = onNavigateToBackup
            ),
            HomeOption(
                icon = Icons.Default.Settings,
                title = "Configuración",
                onClick = onNavigateToSettings
            )
        )
        
        LazyVerticalGrid(
            columns = GridCells.Fixed(2),
            horizontalArrangement = Arrangement.spacedBy(16.dp),
            verticalArrangement = Arrangement.spacedBy(16.dp),
            modifier = Modifier.weight(1f)
        ) {
            items(options) { option ->
                HomeOptionCard(option = option)
            }
        }
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // Botón de cerrar sesión
        OutlinedButton(
            onClick = {
                viewModel.onLogout()
                onLogout()
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Icon(
                imageVector = Icons.Default.Logout,
                contentDescription = null,
                modifier = Modifier.size(20.dp)
            )
            Spacer(modifier = Modifier.width(8.dp))
            Text("Cerrar sesión")
        }
    }
}

@Composable
private fun HomeOptionCard(
    option: HomeOption,
    modifier: Modifier = Modifier
) {
    Card(
        onClick = option.onClick,
        modifier = modifier
            .aspectRatio(1f)
            .fillMaxWidth(),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.Center
        ) {
            Icon(
                imageVector = option.icon,
                contentDescription = null,
                modifier = Modifier.size(48.dp),
                tint = MaterialTheme.colorScheme.primary
            )
            Spacer(modifier = Modifier.height(8.dp))
            Text(
                text = option.title,
                style = MaterialTheme.typography.titleMedium,
                textAlign = TextAlign.Center
            )
        }
    }
}

private data class HomeOption(
    val icon: ImageVector,
    val title: String,
    val onClick: () -> Unit
)
```

## Navegación

```kotlin
composable(Screen.Home.route) {
    HomeScreen(
        viewModel = viewModel,
        onNavigateToPasswords = {
            navController.navigate(Screen.PasswordList.route)
        },
        onNavigateToCategories = {
            navController.navigate(Screen.CategoryManagement.route)
        },
        onNavigateToGenerator = {
            navController.navigate(Screen.PasswordGenerator.route)
        },
        onNavigateToStatistics = {
            navController.navigate(Screen.Statistics.route)
        },
        onNavigateToAudit = {
            navController.navigate(Screen.Audit.route)
        },
        onNavigateToBackup = {
            navController.navigate(Screen.Backup.route)
        },
        onNavigateToSettings = {
            navController.navigate(Screen.Settings.route)
        },
        onLogout = {
            navController.navigate(Screen.Login.route) {
                popUpTo(Screen.Home.route) { inclusive = true }
            }
        }
    )
}
```

## AutoLockManager

La pantalla Home está protegida por `AutoLockManager`. Si el usuario sale de la app y vuelve después del timeout configurado, se redirige automáticamente a Login.

```kotlin
// En MainActivity
if (autoLockManager.isLocked()) {
    // Redirigir a Login
    navController.navigate(Screen.Login.route) {
        popUpTo(Screen.Home.route) { inclusive = true }
    }
}
```

## Referencias

- [AutoLockManager](../presentation/overview.md#autolockmanager)
- [NavGraph](../presentation/overview.md#navegación-detallada)
- [AuthViewModel](../viewmodels/overview.md#authviewmodel)
