# Navegación (NavGraph)

## Visión General

La navegación utiliza Navigation Compose con un solo grafo de navegación que maneja todos los flujos de la aplicación.

**Archivo**: `presentation/navigation/NavGraph.kt`

## Rutas Disponibles

```kotlin
sealed class Screen(val route: String) {
    object Onboarding : Screen("onboarding")
    object Login : Screen("login")
    object Home : Screen("home")
    object PasswordList : Screen("passwords")
    object PasswordDetail : Screen("passwords/{entryId}") {
        fun createRoute(entryId: String) = "passwords/$entryId"
    }
    object PasswordCreate : Screen("passwords/create")
    object PasswordEdit : Screen("passwords/{entryId}/edit") {
        fun createRoute(entryId: String) = "passwords/$entryId/edit"
    }
    object CategoryManagement : Screen("settings/categories")
    object Backup : Screen("settings/backup")
    object Settings : Screen("settings")
    object ChangePassword : Screen("settings/change-password")
    object Audit : Screen("tools/audit")
    object Statistics : Screen("tools/statistics")
}
```

## Determinación de Pantalla Inicial

```kotlin
private fun determineStartDestination(
    isSetupComplete: Boolean,
    isAuthenticated: Boolean
): String {
    return when {
        !isSetupComplete -> Screen.Onboarding.route
        !isAuthenticated -> Screen.Login.route
        else -> Screen.Home.route
    }
}
```

## Flujo de Navegación

```
┌─────────────────────────────────────────────────────────────┐
│                    App Start                                 │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
              ┌─────────────────────────┐
              │  ¿Setup Completado?     │
              └─────────────────────────┘
                    │            │
                   NO           SÍ
                    │            │
                    ▼            ▼
           ┌────────────┐  ┌────────────┐
           │ Onboarding │  │ ¿Auth?     │
           └────────────┘  └────────────┘
                                │     │
                               NO    SÍ
                                │     │
                                ▼     ▼
                         ┌────────┐ ┌──────┐
                         │ Login  │ │ Home │
                         └────────┘ └──────┘
                              │           │
                              │           ├──→ PasswordList
                              │           ├──→ CategoryManagement
                              │           ├──→ Generator
                              │           ├──→ Statistics
                              │           ├──→ Audit
                              │           ├──→ Backup
                              │           └──→ Settings
                              │                │
                              │                └──→ ChangePassword
                              │
                              ▼ (success)
                           Home
```

## Implementación del NavHost

```kotlin
@Composable
fun NavGraph(
    navController: NavHostController = rememberNavController()
) {
    val viewModel: AuthViewModel = koinViewModel()
    val state by viewModel.state.collectAsStateWithLifecycle()
    
    var startDestination by remember { mutableStateOf<String?>(null) }
    
    val isSetupComplete = state.isSetupComplete
    val isAuthenticated = state.isAuthenticated
    
    LaunchedEffect(isSetupComplete, isAuthenticated) {
        if (startDestination == null) {
            startDestination = determineStartDestination(
                isSetupComplete,
                isAuthenticated
            )
        }
    }
    
    startDestination?.let { destination ->
        Scaffold { innerPadding ->
            NavHost(
                navController = navController,
                startDestination = destination,
                modifier = Modifier.padding(innerPadding)
            ) {
                // Onboarding
                composable(Screen.Onboarding.route) {
                    OnboardingScreen(
                        viewModel = viewModel,
                        onSetupComplete = {
                            navController.navigate(Screen.Login.route) {
                                popUpTo(Screen.Onboarding.route) { inclusive = true }
                            }
                        }
                    )
                }
                
                // Login
                composable(Screen.Login.route) {
                    LoginScreen(
                        viewModel = viewModel,
                        onLoginSuccess = {
                            navController.navigate(Screen.Home.route) {
                                popUpTo(Screen.Login.route) { inclusive = true }
                            }
                        }
                    )
                }
                
                // Home
                composable(Screen.Home.route) {
                    HomeScreen(
                        viewModel = viewModel,
                        onNavigateToPasswords = {
                            navController.navigate(Screen.PasswordList.route)
                        },
                        // ... más navegaciones
                        onLogout = {
                            navController.navigate(Screen.Login.route) {
                                popUpTo(Screen.Home.route) { inclusive = true }
                            }
                        }
                    )
                }
                
                // PasswordList
                composable(Screen.PasswordList.route) {
                    PasswordListScreen(
                        onNavigateToDetail = { entryId ->
                            navController.navigate(Screen.PasswordDetail.createRoute(entryId))
                        },
                        onNavigateToCreate = {
                            navController.navigate(Screen.PasswordCreate.route)
                        },
                        onNavigateToEdit = { entryId ->
                            navController.navigate(Screen.PasswordEdit.createRoute(entryId))
                        }
                    )
                }
                
                // PasswordDetail (con argumento)
                composable(
                    route = Screen.PasswordDetail.route,
                    arguments = listOf(
                        navArgument("entryId") { type = NavType.StringType }
                    )
                ) { backStackEntry ->
                    val entryId = backStackEntry.arguments?.getString("entryId")
                        ?: return@composable
                    PasswordDetailScreen(
                        entryId = entryId,
                        onNavigateBack = { navController.popBackStack() },
                        onNavigateToEdit = { editId ->
                            navController.navigate(Screen.PasswordEdit.createRoute(editId))
                        }
                    )
                }
                
                // PasswordCreate
                composable(Screen.PasswordCreate.route) {
                    PasswordFormScreen(
                        onNavigateBack = { navController.popBackStack() },
                        onSaveComplete = { navController.popBackStack() }
                    )
                }
                
                // PasswordEdit (con argumento)
                composable(
                    route = Screen.PasswordEdit.route,
                    arguments = listOf(
                        navArgument("entryId") { type = NavType.StringType }
                    )
                ) { backStackEntry ->
                    val entryId = backStackEntry.arguments?.getString("entryId")
                        ?: return@composable
                    PasswordFormScreen(
                        editEntryId = entryId,
                        onNavigateBack = { navController.popBackStack() },
                        onSaveComplete = { navController.popBackStack() }
                    )
                }
                
                // CategoryManagement
                composable(Screen.CategoryManagement.route) {
                    CategoryManagementScreen(
                        onNavigateBack = { navController.popBackStack() }
                    )
                }
                
                // Backup
                composable(Screen.Backup.route) {
                    BackupScreen(
                        onNavigateBack = { navController.popBackStack() }
                    )
                }
                
                // Audit
                composable(Screen.Audit.route) {
                    AuditScreen()
                }
                
                // Statistics
                composable(Screen.Statistics.route) {
                    StatisticsScreen()
                }
                
                // Settings
                composable(Screen.Settings.route) {
                    SettingsScreen(
                        onNavigateToChangePassword = {
                            navController.navigate(Screen.ChangePassword.route)
                        }
                    )
                }
                
                // ChangePassword
                composable(Screen.ChangePassword.route) {
                    ChangePasswordScreen(
                        onNavigateBack = { navController.popBackStack() },
                        onPasswordChanged = {
                            navController.popBackStack()
                        }
                    )
                }
            }
        }
    }
}
```

## Patrones de Navegación

### Navegación con PopUpTo (evitar backstack duplicado)

```kotlin
// Login → Home (eliminar Login del backstack)
navController.navigate(Screen.Home.route) {
    popUpTo(Screen.Login.route) { inclusive = true }
}

// Onboarding → Login (eliminar Onboarding del backstack)
navController.navigate(Screen.Login.route) {
    popUpTo(Screen.Onboarding.route) { inclusive = true }
}
```

### Navegación con Argumentos

```kotlin
// Definir ruta con argumento
object PasswordDetail : Screen("passwords/{entryId}") {
    fun createRoute(entryId: String) = "passwords/$entryId"
}

// Navegar con argumento
navController.navigate(Screen.PasswordDetail.createRoute(entryId))

// Leer argumento en el destino
composable(
    route = Screen.PasswordDetail.route,
    arguments = listOf(
        navArgument("entryId") { type = NavType.StringType }
    )
) { backStackEntry ->
    val entryId = backStackEntry.arguments?.getString("entryId")
    // ...
}
```

### Volver Atrás

```kotlin
// Simple popBackStack
onNavigateBack = { navController.popBackStack() }

// Pop hasta una ruta específica
onSaveComplete = {
    navController.popBackStack(Screen.PasswordList.route, inclusive = false)
}
```

## AutoLockManager y Navegación

El `AutoLockManager` puede forzar navegación a Login cuando la app vuelve de background después del timeout:

```kotlin
// En MainActivity
if (autoLockManager.isLocked()) {
    navController.navigate(Screen.Login.route) {
        popUpTo(Screen.Home.route) { inclusive = true }
    }
}
```

## Referencias

- [Navigation Compose](https://developer.android.com/jetpack/compose/navigation)
- [NavGraph](../presentation/overview.md#navegación-detallada)
