# SettingsScreen

## Visión General

Pantalla de configuración de la aplicación donde el usuario puede personalizar el tema, la biometría, el timeout de bloqueo y otras opciones.

**Archivo**: `presentation/ui/screens/SettingsScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Configuración                        │
├─────────────────────────────────────────┤
│                                         │
│  APARIENCIA                             │
│  ┌─────────────────────────────────┐    │
│  │ Tema         [Auto ▼]           │    │
│  └─────────────────────────────────┘    │
│                                         │
│  SEGURIDAD                              │
│  ┌─────────────────────────────────┐    │
│  │ Autenticación biométrica  [On] │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │ Auto-bloqueo     [5 min ▼]     │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │ Cambiar contraseña maestra  >  │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ACERCA DE                              │
│  ┌─────────────────────────────────┐    │
│  │ Versión            1.0.0       │    │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

## Estado de la UI

```kotlin
data class SettingsState(
    val lockTimeout: Int = 5,           // minutos
    val biometricEnabled: Boolean = true,
    val themeMode: ThemeMode = ThemeMode.SYSTEM,
    val showTimeoutPicker: Boolean = false,
    val showThemePicker: Boolean = false,
    val isLoading: Boolean = false,
    val errorMessage: String? = null,
    val successMessage: String? = null
)
```

## ThemeMode

```kotlin
enum class ThemeMode(val value: Int, val displayName: String) {
    SYSTEM(0, "Auto"),
    LIGHT(1, "Claro"),
    DARK(2, "Oscuro");
    
    companion object {
        fun fromInt(value: Int): ThemeMode = entries.find { it.value == value } ?: SYSTEM
    }
}
```

## Código del Componente

```kotlin
@Composable
fun SettingsScreen(
    viewModel: SettingsViewModel = koinViewModel(),
    onNavigateToChangePassword: () -> Unit,
    modifier: Modifier = Modifier
) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Configuración") },
                navigationIcon = {
                    IconButton(onClick = { /* Back */ }) {
                        Icon(Icons.Default.ArrowBack, null)
                    }
                }
            )
        }
    ) { padding ->
        Column(
            modifier = modifier
                .fillMaxSize()
                .padding(padding)
        ) {
            // Sección: Apariencia
            Text(
                text = "APARIENCIA",
                style = MaterialTheme.typography.labelLarge,
                color = MaterialTheme.colorScheme.primary,
                modifier = Modifier.padding(16.dp)
            )
            
            SettingsItem(
                title = "Tema",
                value = state.themeMode.displayName,
                onClick = { viewModel.showThemePicker() }
            )
            
            Divider()
            
            // Sección: Seguridad
            Text(
                text = "SEGURIDAD",
                style = MaterialTheme.typography.labelLarge,
                color = MaterialTheme.colorScheme.primary,
                modifier = Modifier.padding(16.dp)
            )
            
            SettingsSwitchItem(
                title = "Autenticación biométrica",
                subtitle = "Usar huella o rostro para desbloquear",
                checked = state.biometricEnabled,
                onCheckedChange = { viewModel.toggleBiometric() }
            )
            
            Divider()
            
            SettingsItem(
                title = "Auto-bloqueo",
                value = "${state.lockTimeout} minutos",
                onClick = { viewModel.showTimeoutPicker() }
            )
            
            Divider()
            
            SettingsItem(
                title = "Cambiar contraseña maestra",
                onClick = onNavigateToChangePassword
            )
            
            Divider()
            
            // Sección: Acerca de
            Text(
                text = "ACERCA DE",
                style = MaterialTheme.typography.labelLarge,
                color = MaterialTheme.colorScheme.primary,
                modifier = Modifier.padding(16.dp)
            )
            
            SettingsItem(
                title = "Versión",
                value = "1.0.0",
                enabled = false
            )
        }
        
        // Dialogs
        if (state.showThemePicker) {
            ThemePickerDialog(
                currentMode = state.themeMode,
                onThemeSelected = { viewModel.onEvent(SettingsEvent.OnThemeModeChanged(it)) },
                onDismiss = { viewModel.onEvent(SettingsEvent.DismissThemePicker) }
            )
        }
        
        if (state.showTimeoutPicker) {
            TimeoutPickerDialog(
                currentTimeout = state.lockTimeout,
                onTimeoutSelected = { 
                    viewModel.onEvent(SettingsEvent.OnLockTimeoutChanged(it))
                },
                onDismiss = { viewModel.onEvent(SettingsEvent.DismissTimeoutPicker) }
            )
        }
        
        // Snackbar para mensajes
        state.successMessage?.let { message ->
            Snackbar(
                modifier = Modifier.padding(16.dp),
                action = {
                    TextButton(onClick = { 
                        viewModel.onEvent(SettingsEvent.DismissMessage)
                    }) {
                        Text("OK")
                    }
                }
            ) {
                Text(message)
            }
        }
    }
}

@Composable
private fun SettingsItem(
    title: String,
    value: String? = null,
    enabled: Boolean = true,
    onClick: () -> Unit = {}
) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable(enabled = enabled, onClick = onClick)
            .padding(16.dp),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            text = title,
            style = MaterialTheme.typography.bodyLarge
        )
        Row(verticalAlignment = Alignment.CenterVertically) {
            value?.let {
                Spacer(modifier = Modifier.width(8.dp))
                Text(
                    text = it,
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
            Icon(
                imageVector = Icons.Default.ChevronRight,
                contentDescription = null,
                tint = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
    }
}

@Composable
private fun SettingsSwitchItem(
    title: String,
    subtitle: String? = null,
    checked: Boolean,
    onCheckedChange: (Boolean) -> Unit
) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable { onCheckedChange(!checked) }
            .padding(16.dp),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Column(modifier = Modifier.weight(1f)) {
            Text(
                text = title,
                style = MaterialTheme.typography.bodyLarge
            )
            subtitle?.let {
                Text(
                    text = it,
                    style = MaterialTheme.typography.bodySmall,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
        Switch(
            checked = checked,
            onCheckedChange = onCheckedChange
        )
    }
}
```

## Diálogos

### ThemePickerDialog

```kotlin
@Composable
fun ThemePickerDialog(
    currentMode: ThemeMode,
    onThemeSelected: (ThemeMode) -> Unit,
    onDismiss: () -> Unit
) {
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Seleccionar tema") },
        text = {
            Column {
                ThemeMode.entries.forEach { mode ->
                    Row(
                        modifier = Modifier
                            .fillMaxWidth()
                            .clickable { 
                                onThemeSelected(mode)
                                onDismiss()
                            }
                            .padding(16.dp)
                    ) {
                        RadioButton(
                            selected = currentMode == mode,
                            onClick = { 
                                onThemeSelected(mode)
                                onDismiss()
                            }
                        )
                        Spacer(modifier = Modifier.width(8.dp))
                        Text(mode.displayName)
                    }
                }
            }
        },
        confirmButton = {
            TextButton(onClick = onDismiss) {
                Text("Cancelar")
            }
        }
    )
}
```

### TimeoutPickerDialog

```kotlin
@Composable
fun TimeoutPickerDialog(
    currentTimeout: Int,
    onTimeoutSelected: (Int) -> Unit,
    onDismiss: () -> Unit
) {
    val options = listOf(1, 2, 5, 10, 15, 30, 60)
    
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Auto-bloqueo (minutos)") },
        text = {
            LazyColumn {
                items(options) { minutes ->
                    Row(
                        modifier = Modifier
                            .fillMaxWidth()
                            .clickable { 
                                onTimeoutSelected(minutes)
                                onDismiss()
                            }
                            .padding(16.dp)
                    ) {
                        RadioButton(
                            selected = currentTimeout == minutes,
                            onClick = { 
                                onTimeoutSelected(minutes)
                                onDismiss()
                            }
                        )
                        Spacer(modifier = Modifier.width(8.dp))
                        Text("$minutes minutos")
                    }
                }
            }
        },
        confirmButton = {
            TextButton(onClick = onDismiss) {
                Text("Cancelar")
            }
        }
    )
}
```

## Navegación

```kotlin
composable(Screen.Settings.route) {
    SettingsScreen(
        viewModel = koinViewModel(),
        onNavigateToChangePassword = {
            navController.navigate(Screen.ChangePassword.route)
        }
    )
}
```

## Referencias

- [SettingsViewModel](../viewmodels/overview.md#settingsviewmodel)
- [ThemeMode](../domain/overview.md#appsettings)
- [AutoLockManager](../presentation/overview.md#autolockmanager)
