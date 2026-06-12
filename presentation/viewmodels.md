# ViewModels

Documentación de todos los ViewModels de la aplicación.

---

## Patrón Base

Todos los ViewModels siguen el patrón:

```kotlin
class ExampleViewModel(
    private val useCase: ExampleUseCase
) : ViewModel() {

    private val _state = MutableStateFlow(ExampleState())
    val state: StateFlow<ExampleState> = _state.asStateFlow()

    fun onEvent(event: UiEvent) {
        viewModelScope.launch {
            // Procesar evento
            _state.value = _state.value.copy(...)
        }
    }
}
```

---

## PasswordListViewModel

Gestiona la lista de contraseñas.

**Archivo**: `viewmodel/PasswordListViewModel.kt`

```kotlin
class PasswordListViewModel(
    private val getAllPasswords: GetAllPasswords,
    private val deletePasswordEntry: DeletePasswordEntry,
    private val toggleFavoriteEntry: ToggleFavoriteEntry
) : ViewModel() {

    private val _searchQuery = MutableStateFlow("")
    private val _selectedCategoryId = MutableStateFlow<String?>(null)

    private val _state = MutableStateFlow(PasswordListState())
    val state: StateFlow<PasswordListState> = _state.asStateFlow()

    init {
        observeFilters()
    }

    private fun observeFilters() {
        viewModelScope.launch {
            combine(
                getAllPasswords(),
                _searchQuery,
                _selectedCategoryId
            ) { entries, query, categoryId ->
                entries.filter { entry ->
                    val matchesSearch = query.isBlank() ||
                        entry.title.contains(query, ignoreCase = true) ||
                        entry.username.contains(query, ignoreCase = true) ||
                        entry.url?.contains(query, ignoreCase = true) == true

                    val matchesCategory = categoryId == null ||
                        entry.categoryId == categoryId

                    matchesSearch && matchesCategory
                }
            }.collect { filtered ->
                _state.value = _state.value.copy(
                    entries = filtered,
                    isLoading = false
                )
            }
        }
    }

    fun onSearchQueryChanged(query: String) {
        _searchQuery.value = query
    }

    fun onCategoryFilterSelected(categoryId: String?) {
        _selectedCategoryId.value = categoryId
    }

    fun clearCategoryFilter() {
        _selectedCategoryId.value = null
    }

    fun onToggleFavorite(entry: PasswordEntry) {
        viewModelScope.launch {
            toggleFavoriteEntry(entry)
        }
    }

    fun onDeleteEntry(entry: PasswordEntry) {
        viewModelScope.launch {
            deletePasswordEntry(entry)
                .onFailure { error ->
                    _state.value = _state.value.copy(
                        error = "Error al eliminar: ${error.message}"
                    )
                }
        }
    }

    fun clearError() {
        _state.value = _state.value.copy(error = null)
    }
}
```

**Estado**:
```kotlin
data class PasswordListState(
    val entries: List<PasswordEntry> = emptyList(),
    val searchQuery: String = "",
    val selectedCategoryId: String? = null,
    val isLoading: Boolean = true,
    val error: String? = null
)
```

**Métodos Públicos**:
| Método | Parámetros | Descripción |
|--------|------------|-------------|
| `onSearchQueryChanged` | `query: String` | Actualiza texto de búsqueda |
| `onCategoryFilterSelected` | `categoryId: String?` | Filtra por categoría |
| `clearCategoryFilter` | - | Resetea filtro de categoría |
| `onToggleFavorite` | `entry: PasswordEntry` | Alterna favorito |
| `onDeleteEntry` | `entry: PasswordEntry` | Elimina contraseña |
| `clearError` | - | Limpia mensaje de error |

---

## PasswordFormViewModel

Gestiona el formulario de creación/edición.

**Archivo**: `viewmodel/PasswordFormViewModel.kt`

```kotlin
class PasswordFormViewModel(
    private val createCategory: CreateCategory,
    private val updateCategory: UpdateCategory,
    private val deleteCategory: DeleteCategory,
    private val getCategories: GetCategories,
    private val createPasswordEntry: CreatePasswordEntry
) : ViewModel() {

    private val _state = MutableStateFlow(PasswordFormState())
    val state: StateFlow<PasswordFormState> = _state.asStateFlow()

    private val _categories = MutableStateFlow<List<Category>>(emptyList())
    val categories: StateFlow<List<Category>> = _categories.asStateFlow()

    init {
        loadCategories()
    }

    private fun loadCategories() {
        viewModelScope.launch {
            getCategories().collect { categories ->
                _categories.value = categories
                // Seleccionar "General" por defecto
                if (_state.value.selectedCategory == null) {
                    _state.value = _state.value.copy(
                        selectedCategory = categories.find { it.name == "General" }
                    )
                }
            }
        }
    }

    fun onFieldChange(field: FormField, value: String) {
        _state.value = when (field) {
            FormField.TITLE -> _state.value.copy(title = value)
            FormField.USERNAME -> _state.value.copy(username = value)
            FormField.PASSWORD -> _state.value.copy(password = value)
            FormField.URL -> _state.value.copy(url = value)
            FormField.NOTES -> _state.value.copy(notes = value)
        }
    }

    fun onCategorySelected(category: Category) {
        _state.value = _state.value.copy(selectedCategory = category)
    }

    fun onToggleFavorite() {
        _state.value = _state.value.copy(
            isFavorite = !_state.value.isFavorite
        )
    }

    fun onSave() {
        viewModelScope.launch {
            if (!validateForm()) return@launch

            val entry = PasswordEntry(
                id = UUID.randomUUID().toString(),
                title = _state.value.title,
                username = _state.value.username,
                password = _state.value.password,
                url = _state.value.url,
                notes = _state.value.notes,
                categoryId = _state.value.selectedCategory?.id ?: "",
                icon = null,
                isFavorite = _state.value.isFavorite,
                createdAt = System.currentTimeMillis(),
                updatedAt = System.currentTimeMillis()
            )

            createPasswordEntry(entry)
                .onSuccess { /* Navegar atrás */ }
                .onFailure { error ->
                    _state.value = _state.value.copy(
                        error = error.message ?: "Error al guardar"
                    )
                }
        }
    }

    private fun validateForm(): Boolean {
        return when {
            _state.value.title.isBlank() -> {
                _state.value = _state.value.copy(error = "El título es requerido")
                false
            }
            _state.value.username.isBlank() -> {
                _state.value = _state.value.copy(error = "El username es requerido")
                false
            }
            _state.value.password.isBlank() -> {
                _state.value = _state.value.copy(error = "La contraseña es requerida")
                false
            }
            else -> true
        }
    }
}
```

**Estado**:
```kotlin
data class PasswordFormState(
    val title: String = "",
    val username: String = "",
    val password: String = "",
    val url: String = "",
    val notes: String = "",
    val selectedCategory: Category? = null,
    val isFavorite: Boolean = false,
    val error: String? = null,
    val isSaving: Boolean = false
)

enum class FormField {
    TITLE, USERNAME, PASSWORD, URL, NOTES
}
```

---

## AuthViewModel

Gestiona autenticación del usuario.

**Archivo**: `viewmodel/AuthViewModel.kt`

```kotlin
class AuthViewModel(
    private val setupMasterPassword: SetupMasterPassword,
    private val authenticateUser: AuthenticateUser,
    private val seedPredefinedCategories: SeedPredefinedCategories,
    private val authRepository: AuthRepository,
    private val autoLockManager: AutoLockManager,
    private val application: Application
) : ViewModel() {

    private val _state = MutableStateFlow(LoginState())
    val state: StateFlow<LoginState> = _state.asStateFlow()

    init {
        checkBiometricAvailability()
    }

    private fun checkBiometricAvailability() {
        val biometricAuthenticator = BiometricAuthenticator()
        val status = biometricAuthenticator.canAuthenticate(application)
        _state.value = _state.value.copy(
            isBiometricAvailable = status == BiometricStatus.Available
        )
    }

    fun onPasswordChange(password: String) {
        _state.value = _state.value.copy(password = password)
    }

    fun onLoginClick() {
        viewModelScope.launch {
            _state.value = _state.value.copy(isLoading = true)

            val result = authenticateUser(_state.value.password.toCharArray())

            result
                .onSuccess { masterKey ->
                    authRepository.setMasterKey(masterKey)
                    autoLockManager.resetTimer()
                    _state.value = _state.value.copy(
                        isLoading = false,
                        error = null
                    )
                    // Navegar a Home
                }
                .onFailure { error ->
                    _state.value = _state.value.copy(
                        isLoading = false,
                        error = error.message ?: "Error de autenticación"
                    )
                }
        }
    }

    fun onBiometricAuthSuccess() {
        viewModelScope.launch {
            // Completar flujo de autenticación
        }
    }
}
```

**Estado**:
```kotlin
data class LoginState(
    val password: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
    val isBiometricAvailable: Boolean = false
)
```

---

## PasswordDetailViewModel

Gestiona la pantalla de detalle de una contraseña individual.

**Archivo**: `viewmodel/PasswordDetailViewModel.kt`

```kotlin
class PasswordDetailViewModel(
    private val getPasswordById: GetPasswordById,
    private val context: Context
) : ViewModel() {

    private val clipboardManager: ClipboardManager by lazy {
        context.getSystemService(Context.CLIPBOARD_SERVICE) as ClipboardManager
    }

    private val _state = MutableStateFlow(PasswordDetailState())
    val state: StateFlow<PasswordDetailState> = _state.asStateFlow()

    fun loadEntry(entryId: String) {
        viewModelScope.launch {
            val entry = getPasswordById(entryId)
            _state.value = _state.value.copy(
                entry = entry,
                isLoading = false,
                error = if (entry == null) "Entrada no encontrada" else null
            )
        }
    }

    fun togglePasswordVisibility() {
        _state.value = _state.value.copy(
            showPassword = !_state.value.showPassword
        )
    }

    fun copyUsername(): ClipboardResult {
        val username = _state.value.entry?.username
        if (username.isNullOrBlank()) {
            return ClipboardResult(false, "No hay usuario para copiar")
        }
        return copyToClipboard(username, "Usuario copiado")
    }

    fun copyPassword(): ClipboardResult {
        val password = _state.value.entry?.password
        if (password.isNullOrBlank()) {
            return ClipboardResult(false, "No hay contraseña para copiar")
        }
        return copyToClipboard(password, "Contraseña copiada")
    }

    private fun copyToClipboard(text: String, successMessage: String): ClipboardResult {
        val clip = ClipData.newPlainText("Password Manager", text)
        clipboardManager.setPrimaryClip(clip)
        return ClipboardResult(true, successMessage)
    }
}
```

**Estado**:
```kotlin
data class PasswordDetailState(
    val entry: PasswordEntry? = null,
    val showPassword: Boolean = false,
    val isLoading: Boolean = true,
    val error: String? = null
)
```

---

## PasswordGeneratorViewModel

Gestiona la generación de contraseñas seguras.

**Archivo**: `viewmodel/PasswordGeneratorViewModel.kt`

```kotlin
class PasswordGeneratorViewModel(
    private val generatePassword: GeneratePassword
) : ViewModel() {

    var state by mutableStateOf(PasswordGeneratorState())
        private set

    init {
        generatePassword()  // Generar una al iniciar
    }

    fun onEvent(event: PasswordGeneratorEvent) {
        when (event) {
            is PasswordGeneratorEvent.OnLengthChanged -> {
                state = state.copy(passwordLength = event.length)
                generatePassword()
            }
            is PasswordGeneratorEvent.OnUseUppercaseChanged -> {
                state = state.copy(useUppercase = event.use)
            }
            is PasswordGeneratorEvent.OnGenerateClicked -> generatePassword()
            is PasswordGeneratorEvent.OnCopyClicked -> {
                state = state.copy(showCopiedMessage = true)
            }
        }
    }

    private fun generatePassword() {
        val options = GeneratePassword.PasswordOptions(
            length = state.passwordLength,
            useUppercase = state.useUppercase,
            useLowercase = state.useLowercase,
            useDigits = state.useDigits,
            useSymbols = state.useSymbols,
            excludeAmbiguous = state.excludeAmbiguous
        )

        val password = generatePassword(options)
        val strengthScore = generatePassword.calculateStrength(password)

        state = state.copy(
            generatedPassword = password,
            strengthScore = strengthScore,
            strengthLabel = generatePassword.getStrengthLabel(strengthScore),
            strengthColor = generatePassword.getStrengthColor(strengthScore)
        )
    }
}
```

**Estado**:
```kotlin
data class PasswordGeneratorState(
    val generatedPassword: String = "",
    val passwordLength: Int = 16,
    val useUppercase: Boolean = true,
    val useLowercase: Boolean = true,
    val useDigits: Boolean = true,
    val useSymbols: Boolean = true,
    val excludeAmbiguous: Boolean = false,
    val strengthScore: Int = 0,
    val strengthLabel: String = "Débil",
    val strengthColor: Long = 0xFFFF0000L,
    val showCopiedMessage: Boolean = false
)
```

**Cálculo de Fortaleza**:
```kotlin
fun calculateStrength(password: String): Int {
    var score = 0
    if (password.length >= 8) score += 20
    if (password.length >= 12) score += 20
    if (password.length >= 16) score += 20
    if (password.length >= 20) score += 10
    if (password.any { it.isUpperCase() }) score += 15
    if (password.any { it.isLowerCase() }) score += 10
    if (password.any { it.isDigit() }) score += 15
    if (password.any { !it.isLetterOrDigit() }) score += 10
    return score.coerceIn(0, 100)
}
```

---

## SettingsViewModel

Gestiona la configuración de la aplicación.

**Archivo**: `viewmodel/SettingsViewModel.kt`

```kotlin
class SettingsViewModel(
    private val getSettings: GetSettings,
    private val updateSettings: UpdateSettings,
    private val setLockTimeout: SetLockTimeout,
    private val setBiometricEnabled: SetBiometricEnabled,
    private val setThemeMode: SetThemeMode
) : ViewModel() {

    private val _state = MutableStateFlow(SettingsState())
    val state: StateFlow<SettingsState> = _state.asStateFlow()

    init {
        loadSettings()
    }

    private fun loadSettings() {
        getSettings()
            .onEach { settings ->
                _state.value = _state.value.copy(
                    lockTimeout = settings.lockTimeoutMinutes,
                    biometricEnabled = settings.biometricEnabled,
                    themeMode = settings.themeMode
                )
            }
            .catch { e ->
                _state.value = _state.value.copy(
                    errorMessage = "Error al cargar configuración: ${e.message}"
                )
            }
            .launchIn(viewModelScope)
    }

    fun updateLockTimeout(minutes: Int) {
        viewModelScope.launch {
            setLockTimeout(minutes)
                .onSuccess {
                    _state.value = _state.value.copy(
                        lockTimeout = minutes,
                        successMessage = "Auto-bloqueo actualizado"
                    )
                }
                .onFailure { e ->
                    _state.value = _state.value.copy(
                        errorMessage = "Error: ${e.message}"
                    )
                }
        }
    }

    fun toggleBiometric() {
        viewModelScope.launch {
            val newValue = !_state.value.biometricEnabled
            setBiometricEnabled(newValue)
                .onSuccess {
                    _state.value = _state.value.copy(
                        biometricEnabled = newValue,
                        successMessage = if (newValue) "Biometría activada" else "Desactivada"
                    )
                }
        }
    }
}
```

**Estado**:
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

---

## AuditViewModel

Gestiona la auditoría de contraseñas débiles.

**Archivo**: `viewmodel/AuditViewModel.kt`

```kotlin
class AuditViewModel(
    private val auditWeakPasswords: AuditWeakPasswords,
    private val passwordRepository: PasswordRepository
) : ViewModel() {

    private val _state = MutableStateFlow(AuditState())
    val state: StateFlow<AuditState> = _state.asStateFlow()

    init {
        runAudit()
    }

    fun runAudit() {
        viewModelScope.launch {
            _state.value = _state.value.copy(isLoading = true)

            val result = auditWeakPasswords()

            _state.value = _state.value.copy(
                isLoading = false,
                totalPasswords = result.totalEntries,
                weakPasswords = result.weakCount,
                weakEntries = result.weakEntries,
                securityScore = calculateSecurityScore(result)
            )
        }
    }

    private fun calculateSecurityScore(result: AuditResult): Int {
        val weakPercentage = result.weakCount.toFloat() / result.totalEntries
        return ((1 - weakPercentage) * 100).toInt().coerceIn(0, 100)
    }
}
```

**Estado**:
```kotlin
data class AuditState(
    val totalPasswords: Int = 0,
    val weakPasswords: Int = 0,
    val weakEntries: List<WeakPasswordEntry> = emptyList(),
    val securityScore: Int = 0,
    val isLoading: Boolean = false,
    val error: String? = null
)
```

---

## StatisticsViewModel

Gestiona las estadísticas de seguridad.

**Archivo**: `viewmodel/StatisticsViewModel.kt`

```kotlin
class StatisticsViewModel(
    private val getSecurityStatistics: GetSecurityStatistics
) : ViewModel() {

    private val _state = MutableStateFlow(StatisticsState())
    val state: StateFlow<StatisticsState> = _state.asStateFlow()

    init {
        loadStatistics()
    }

    fun loadStatistics() {
        viewModelScope.launch {
            _state.value = _state.value.copy(isLoading = true)

            val stats = getSecurityStatistics()

            _state.value = _state.value.copy(
                isLoading = false,
                totalPasswords = stats.totalPasswords,
                favoriteCount = stats.favoriteCount,
                categoryCount = stats.categoryCount,
                weakPasswordCount = stats.weakPasswordCount,
                reusedPasswordCount = stats.reusedPasswordCount,
                oldPasswordCount = stats.oldPasswordCount,
                averageStrength = stats.averageStrength
            )
        }
    }
}
```

**Estado**:
```kotlin
data class StatisticsState(
    val totalPasswords: Int = 0,
    val favoriteCount: Int = 0,
    val categoryCount: Int = 0,
    val weakPasswordCount: Int = 0,
    val reusedPasswordCount: Int = 0,
    val oldPasswordCount: Int = 0,
    val averageStrength: Double = 0.0,
    val isLoading: Boolean = false,
    val error: String? = null
)
```

---

## BackupViewModel

Gestiona exportación e importación de backups.

**Archivo**: `viewmodel/BackupViewModel.kt`

```kotlin
class BackupViewModel(
    private val exportEncryptedBackup: ExportEncryptedBackup,
    private val importFromCSV: ImportFromCSV,
    private val authRepository: AuthRepository
) : ViewModel() {

    private val _state = MutableStateFlow(BackupState())
    val state: StateFlow<BackupState> = _state.asStateFlow()

    fun exportBackup(outputPath: String) {
        viewModelScope.launch {
            _state.value = _state.value.copy(isLoading = true)

            if (authRepository.getMasterKey() == null) {
                _state.value = _state.value.copy(
                    isLoading = false,
                    error = "No hay sesión activa"
                )
                return@launch
            }

            exportEncryptedBackup(outputPath)
                .onSuccess {
                    _state.value = _state.value.copy(
                        isLoading = false,
                        successMessage = "Backup exportado correctamente"
                    )
                }
                .onFailure { e ->
                    _state.value = _state.value.copy(
                        isLoading = false,
                        error = "Error al exportar: ${e.message}"
                    )
                }
        }
    }

    fun importBackup(inputPath: String) {
        viewModelScope.launch {
            _state.value = _state.value.copy(isLoading = true)

            importFromCSV(inputPath)
                .onSuccess { count ->
                    _state.value = _state.value.copy(
                        isLoading = false,
                        successMessage = "$count contraseñas importadas"
                    )
                }
                .onFailure { e ->
                    _state.value = _state.value.copy(
                        isLoading = false,
                        error = "Error al importar: ${e.message}"
                    )
                }
        }
    }
}
```

**Estado**:
```kotlin
data class BackupState(
    val isLoading: Boolean = false,
    val isExporting: Boolean = false,
    val isImporting: Boolean = false,
    val progress: Float = 0f,
    val successMessage: String? = null,
    val error: String? = null
)
```

---

## ChangePasswordViewModel

Gestiona el cambio de contraseña maestra.

**Archivo**: `viewmodel/ChangePasswordViewModel.kt`

```kotlin
class ChangePasswordViewModel(
    private val changeMasterPassword: ChangeMasterPassword,
    private val authRepository: AuthRepository
) : ViewModel() {

    private val _state = MutableStateFlow(ChangePasswordState())
    val state: StateFlow<ChangePasswordState> = _state.asStateFlow()

    fun onChangePassword(
        oldPassword: CharArray,
        newPassword: CharArray,
        confirmPassword: CharArray
    ) {
        viewModelScope.launch {
            // Validaciones
            if (!newPassword.contentEquals(confirmPassword)) {
                _state.value = _state.value.copy(
                    error = "Las contraseñas no coinciden"
                )
                return@launch
            }

            if (!PasswordValidator.isValid(newPassword)) {
                _state.value = _state.value.copy(
                    error = "La contraseña no cumple los requisitos"
                )
                return@launch
            }

            _state.value = _state.value.copy(isLoading = true)

            changeMasterPassword(oldPassword, newPassword)
                .onSuccess {
                    authRepository.clearMasterKey()
                    _state.value = _state.value.copy(
                        isLoading = false,
                        successMessage = "Contraseña cambiada. Inicia sesión nuevamente."
                    )
                }
                .onFailure { e ->
                    _state.value = _state.value.copy(
                        isLoading = false,
                        error = "Error: ${e.message}"
                    )
                }
        }
    }
}
```

**Estado**:
```kotlin
data class ChangePasswordState(
    val isLoading: Boolean = false,
    val oldPassword: String = "",
    val newPassword: String = "",
    val confirmPassword: String = "",
    val error: String? = null,
    val successMessage: String? = null
)
```
