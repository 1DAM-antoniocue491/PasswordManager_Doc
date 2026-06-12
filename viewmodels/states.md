# Estados de UI (UI States)

## Visión General

Los estados de UI son data classes inmutables que representan el estado de cada pantalla. Siguen el patrón MVI (Model-View-Intent).

## Estructura Común

```kotlin
data class ScreenState(
    val isLoading: Boolean = false,
    val error: String? = null,
    val successMessage: String? = null,
    // ... más propiedades específicas
)
```

## Estados por Pantalla

### PasswordListState

```kotlin
data class PasswordListState(
    val entries: List<PasswordEntry> = emptyList(),
    val filteredEntries: List<PasswordEntry> = emptyList(),
    val favorites: List<PasswordEntry> = emptyList(),
    val categories: List<Category> = emptyList(),
    val selectedCategory: Category? = null,
    val searchQuery: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
    val showDeleteConfirmation: Boolean = false,
    val entryToDelete: PasswordEntry? = null
)
```

### PasswordFormState

```kotlin
data class PasswordFormState(
    val title: String = "",
    val username: String = "",
    val password: String = "",
    val url: String = "",
    val notes: String = "",
    val categoryId: String? = null,
    val isFavorite: Boolean = false,
    val showPassword: Boolean = false,
    val showGenerator: Boolean = false,
    val showCategoryPicker: Boolean = false,
    val isLoading: Boolean = false,
    val error: String? = null,
    val isEdit: Boolean = false,
    val editEntryId: String? = null
)
```

### PasswordDetailState

```kotlin
data class PasswordDetailState(
    val entry: PasswordEntry? = null,
    val showPassword: Boolean = false,
    val isLoading: Boolean = true,
    val error: String? = null,
    val showDeleteConfirmation: Boolean = false,
    val copiedField: String? = null  // "username", "password", null
)
```

### LoginState

```kotlin
data class LoginState(
    val password: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
    val isBiometricAvailable: Boolean = false,
    val showBiometricPrompt: Boolean = false,
    val isSetupComplete: Boolean = false,
    val isAuthenticated: Boolean = false
)
```

### SettingsState

```kotlin
data class SettingsState(
    val lockTimeout: Int = 5,
    val biometricEnabled: Boolean = true,
    val themeMode: ThemeMode = ThemeMode.SYSTEM,
    val showTimeoutPicker: Boolean = false,
    val showThemePicker: Boolean = false,
    val isLoading: Boolean = false,
    val errorMessage: String? = null,
    val successMessage: String? = null
)
```

### BackupState

```kotlin
data class BackupState(
    val isLoading: Boolean = false,
    val isExporting: Boolean = false,
    val isImporting: Boolean = false,
    val progress: Float = 0f,
    val successMessage: String? = null,
    val error: String? = null,
    val exportPath: String? = null,
    val importPath: String? = null
)
```

### AuditState

```kotlin
data class AuditState(
    val totalPasswords: Int = 0,
    val weakPasswords: Int = 0,
    val weakEntries: List<WeakPasswordEntry> = emptyList(),
    val securityScore: Int = 0,
    val isLoading: Boolean = false,
    val error: String? = null
)

data class WeakPasswordEntry(
    val id: String,
    val title: String,
    val reason: String  // "Menos de 8 caracteres", "Sin números", etc.
)
```

### StatisticsState

```kotlin
data class StatisticsState(
    val totalPasswords: Int = 0,
    val favoriteCount: Int = 0,
    val categoryCount: Int = 0,
    val weakPasswordCount: Int = 0,
    val reusedPasswordCount: Int = 0,
    val oldPasswordCount: Int = 0,
    val averageStrength: Double = 0.0,
    val categoryDistribution: Map<String, Int> = emptyMap(),
    val strengthDistribution: StrengthDistribution = StrengthDistribution(),
    val isLoading: Boolean = false,
    val error: String? = null
)

data class StrengthDistribution(
    val weak: Int = 0,      // 0-40
    val medium: Int = 0,    // 41-70
    val strong: Int = 0,    // 71-90
    val veryStrong: Int = 0 // 91-100
)
```

### ChangePasswordState

```kotlin
data class ChangePasswordState(
    val oldPassword: String = "",
    val newPassword: String = "",
    val confirmPassword: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
    val successMessage: String? = null,
    val showOldPassword: Boolean = false,
    val showNewPassword: Boolean = false
)
```

## Patrones de Uso

### Actualización Inmutable

```kotlin
// ✅ CORRECTO
_state.value = _state.value.copy(
    isLoading = true,
    error = null
)

// ❌ INCORRECTO (no funciona con data class inmutable)
_state.value.isLoading = true
```

### Manejo de Errores

```kotlin
sealed class UiMessage {
    data class Error(val message: String) : UiMessage()
    data class Success(val message: String) : UiMessage()
}

// En el ViewModel
fun dismissMessage() {
    _state.value = _state.value.copy(
        error = null,
        successMessage = null
    )
}
```

### Carga de Datos

```kotlin
fun loadData() {
    viewModelScope.launch {
        _state.value = _state.value.copy(isLoading = true)
        
        useCase()
            .onSuccess { data ->
                _state.value = _state.value.copy(
                    isLoading = false,
                    entries = data
                )
            }
            .onFailure { error ->
                _state.value = _state.value.copy(
                    isLoading = false,
                    error = error.message
                )
            }
    }
}
```

## Referencias

- [ViewModels](./viewmodels/overview.md)
- [PasswordListViewModel](../viewmodels/overview.md#passwordlistviewmodel)
