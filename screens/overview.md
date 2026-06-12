# Pantallas - Visión General

La aplicación consta de 12 pantallas principales que cubren todas las funcionalidades de gestión de contraseñas.

## Lista de Pantallas

| Pantalla | Descripción | Archivo |
|----------|-------------|---------|
| LoginScreen | Autenticación inicial | [login.md](./login.md) |
| OnboardingScreen | Configuración inicial | [onboarding.md](./onboarding.md) |
| HomeScreen | Menú principal | [home.md](./home.md) |
| PasswordListScreen | Lista de contraseñas | [password-list.md](./password-list.md) |
| PasswordDetailScreen | Detalle de contraseña | [password-detail.md](./password-detail.md) |
| PasswordFormScreen | Crear/editar contraseña | [password-form.md](./password-form.md) |
| PasswordGeneratorScreen | Generador de contraseñas | [password-generator.md](./password-generator.md) |
| CategoryManagementScreen | Gestión de categorías | [category-management.md](./category-management.md) |
| SettingsScreen | Configuración | [settings.md](./settings.md) |
| BackupScreen | Copia de seguridad | [backup.md](./backup.md) |
| AuditScreen | Auditoría de seguridad | [audit.md](./audit.md) |
| StatisticsScreen | Estadísticas | [statistics.md](./statistics.md) |
| ChangePasswordScreen | Cambiar contraseña maestra | [change-password.md](./change-password.md) |

## Flujo de Navegación

```
Onboarding → Login → Home
                    │
                    ├──→ PasswordList → PasswordDetail → PasswordForm (edit)
                    │                    │
                    │                    └──→ PasswordForm (create)
                    │
                    ├──→ CategoryManagement
                    │
                    ├──→ PasswordGenerator
                    │
                    ├──→ Statistics
                    │
                    ├──→ Audit
                    │
                    ├──→ Backup
                    │
                    └──→ Settings → ChangePassword
```

## Patrones Comunes

Todas las pantallas siguen el patrón MVVM:

```kotlin
@Composable
fun ScreenName(
    viewModel: ScreenViewModel = koinViewModel(),
    onNavigateBack: () -> Unit,
    onNavigateToOther: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Título de Pantalla") },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(Icons.Default.ArrowBack, null)
                    }
                }
            )
        }
    ) { padding ->
        // Contenido de la pantalla
    }
}
```

## Referencias

- [NavGraph](../presentation/navigation.md)
- [ViewModels](../viewmodels/overview.md)
