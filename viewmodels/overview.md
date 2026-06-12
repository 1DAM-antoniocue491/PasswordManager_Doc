# ViewModels

## Visión General

Los ViewModels son responsables de:
- Gestionar el estado de la UI
- Procesar eventos de usuario
- Coordinar con casos de uso
- Exponer datos reactivos mediante StateFlow

## Arquitectura

```
┌─────────────────────────────────────────────────────────┐
│                      UI (Compose)                        │
│  Observa state.collectAsState()                          │
│  Emite eventos → viewModel.onEvent()                     │
└─────────────────────────────────────────────────────────┘
                           ▲
                           │ StateFlow<State>
                           ▼
┌─────────────────────────────────────────────────────────┐
│                      ViewModel                           │
│  - MutableStateFlow<State>                               │
│  - Funciones onEvent()                                   │
│  - Referencias a Use Cases                               │
└─────────────────────────────────────────────────────────┘
                           ▲
                           │ invoke
                           ▼
┌─────────────────────────────────────────────────────────┐
│                      Use Cases                           │
│  Lógica de negocio específica                            │
└─────────────────────────────────────────────────────────┘
```

## Patrón Base

Todos los ViewModels siguen el patrón:

```kotlin
class ExampleViewModel(
    private val useCase1: ExampleUseCase,
    private val useCase2: AnotherUseCase
) : ViewModel() {

    // Estado privado mutable
    private val _state = MutableStateFlow(ExampleState())
    
    // Estado público inmutable
    val state: StateFlow<ExampleState> = _state.asStateFlow()
    
    // Inicialización
    init {
        loadData()
    }
    
    // Eventos de UI
    fun onEvent(event: UiEvent) {
        viewModelScope.launch {
            // Procesar evento
            _state.value = _state.value.copy(...)
        }
    }
    
    // Limpieza
    override fun onCleared() {
        super.onCleared()
        // Limpiar recursos si es necesario
    }
}
```

## ViewModels Disponibles

| ViewModel | Archivo | Estado Asociado |
|-----------|---------|-----------------|
| `AuthViewModel` | `AuthViewModel.kt` | `LoginState` |
| `PasswordListViewModel` | `PasswordListViewModel.kt` | `PasswordListState` |
| `PasswordFormViewModel` | `PasswordFormViewModel.kt` | `PasswordFormState` |
| `PasswordDetailViewModel` | `PasswordDetailViewModel.kt` | `PasswordDetailState` |
| `PasswordGeneratorViewModel` | `PasswordGeneratorViewModel.kt` | `PasswordGeneratorState` |
| `CategoryManagementViewModel` | `CategoryManagementViewModel.kt` | `CategoryManagementState` |
| `SettingsViewModel` | `SettingsViewModel.kt` | `SettingsState` |
| `BackupViewModel` | `BackupViewModel.kt` | `BackupState` |
| `AuditViewModel` | `AuditViewModel.kt` | `AuditState` |
| `StatisticsViewModel` | `StatisticsViewModel.kt` | `StatisticsState` |
| `ChangePasswordViewModel` | `ChangePasswordViewModel.kt` | `ChangePasswordState` |

---

## PasswordListViewModel

### Propósito

Gestionar la lista de contraseñas con búsqueda y filtrado en tiempo real.

### Dependencias

```kotlin
class PasswordListViewModel(
    private val getAllPasswords: GetAllPasswords,
    private val deletePasswordEntry: DeletePasswordEntry,
    private val toggleFavoriteEntry: ToggleFavoriteEntry
) : ViewModel()
```

### Estado

```kotlin
data class PasswordListState(
    val entries: List[PasswordEntry] = emptyList(),
    val searchQuery: String = "",
    val selectedCategoryId: String? = null,
    val isLoading: Boolean = true,
    val error: String? = null
)
```

### Métodos Públicos

| Método | Parámetros | Descripción |
|--------|------------|-------------|
| `onSearchQueryChanged` | `query: String` | Actualiza búsqueda |
| `onCategoryFilterSelected` | `categoryId: String?` | Filtra por categoría |
| `clearCategoryFilter` | - | Resetear filtros |
| `onToggleFavorite` | `entry: PasswordEntry` | Marcar favorito |
| `onDeleteEntry` | `entry: PasswordEntry` | Eliminar entrada |
| `clearError` | - | Limpiar error |

### Implementación Clave

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
                    isLoading = false,
                    error = null
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

### Flujo Reactivo

```
_searchQuery (MutableStateFlow)
    │
    ├────────────────┐
    │                │
    ▼                ▼
┌─────────────────────────┐
│     combine() + filter  │
└─────────────────────────┘
    │
    ▼
_state.value = copy(entries = filtered)
    │
    ▼
UI se recompone automáticamente
```

---

## PasswordFormViewModel

### Propósito

Gestionar el formulario de creación/edición de contraseñas.

### Dependencias

```kotlin
class PasswordFormViewModel(
    private val createCategory: CreateCategory,
    private val updateCategory: UpdateCategory,
    private val deleteCategory: DeleteCategory,
    private val getCategories: GetCategories,
    private val createPasswordEntry: CreatePasswordEntry
) : ViewModel()
```

### Estado

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

### Métodos Públicos

| Método | Parámetros | Descripción |
|--------|------------|-------------|
| `onFieldChange` | `field: FormField, value: String` | Cambiar campo |
| `onCategorySelected` | `category: Category` | Seleccionar categoría |
| `onToggleFavorite` | - | Toggle favorito |
| `onSave` | - | Guardar entrada |
| `clearError` | - | Limpiar error |

### Implementación Clave

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
        loadCategoriesAndSelectDefault()
    }

    private fun loadCategoriesAndSelectDefault() {
        viewModelScope.launch {
            getCategories().collect { categories ->
                _categories.value = categories
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
            FormField.TITLE -> _state.value.copy(title = value, error = null)
            FormField.USERNAME -> _state.value.copy(username = value, error = null)
            FormField.PASSWORD -> _state.value.copy(password = value, error = null)
            FormField.URL -> _state.value.copy(url = value, error = null)
            FormField.NOTES -> _state.value.copy(notes = value, error = null)
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

            _state.value = _state.value.copy(isSaving = true)

            createPasswordEntry(entry)
                .onSuccess { /* Navegar atrás */ }
                .onFailure { error ->
                    _state.value = _state.value.copy(
                        isSaving = false,
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
            _state.value.selectedCategory == null -> {
                _state.value = _state.value.copy(error = "Debe seleccionar una categoría")
                false
            }
            else -> true
        }
    }
}
```

---

## AuthViewModel

### Propósito

Gestionar autenticación del usuario (login, configuración inicial).

### Dependencias

```kotlin
class AuthViewModel(
    private val setupMasterPassword: SetupMasterPassword,
    private val authenticateUser: AuthenticateUser,
    private val seedPredefinedCategories: SeedPredefinedCategories,
    private val authRepository: AuthRepository,
    private val autoLockManager: AutoLockManager,
    private val application: Application
) : ViewModel()
```

### Estado

```kotlin
data class LoginState(
    val password: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
    val isBiometricAvailable: Boolean = false,
    val hasMasterPassword: Boolean = false
)
```

### Métodos Públicos

| Método | Parámetros | Descripción |
|--------|------------|-------------|
| `onPasswordChange` | `password: String` | Cambiar password |
| `onLoginClick` | - | Iniciar sesión |
| `onBiometricAuthSuccess` | - | Éxito en biometría |
| `onBiometricAuthError` | `error: String` | Error en biometría |
| `clearError` | - | Limpiar error |

---

## Gestión de Errores

### Patrón Común

```kotlin
fun onAction() {
    viewModelScope.launch {
        try {
            useCase()
                .onSuccess { result ->
                    // Éxito
                    _state.value = _state.value.copy(error = null)
                }
                .onFailure { error ->
                    // Error
                    _state.value = _state.value.copy(
                        error = error.message ?: "Error desconocido"
                    )
                }
        } catch (e: Exception) {
            _state.value = _state.value.copy(
                error = e.message ?: "Error inesperado"
            )
        }
    }
}
```

### Tipos de Error

```kotlin
sealed class UiError {
    data class Validation(val message: String) : UiError()
    data class Network(val message: String) : UiError()
    data class Database(val message: String) : UiError()
    object Unauthorized : UiError()
    object Unknown : UiError()
}
```

---

## Testing de ViewModels

### Setup para Tests

```kotlin
class PasswordListViewModelTest {

    private val getAllPasswords = mockk<GetAllPasswords>()
    private val deletePasswordEntry = mockk<DeletePasswordEntry>()
    private val toggleFavoriteEntry = mockk<ToggleFavoriteEntry>()

    private lateinit var viewModel: PasswordListViewModel

    @Before
    fun setup() {
        clearAllMocks()
    }

    @Test
    fun `should emit filtered entries when search query changes`() = runTest {
        // Given
        val entries = listOf(
            PasswordEntry(id = "1", title = "Google", username = "user@gmail.com"),
            PasswordEntry(id = "2", title = "Facebook", username = "user@fb.com")
        )
        coEvery { getAllPasswords() } returns flowOf(entries)

        // When
        viewModel = PasswordListViewModel(getAllPasswords, deletePasswordEntry, toggleFavoriteEntry)
        viewModel.onSearchQueryChanged("gmail")

        // Then
        val state = viewModel.state.first()
        assertEquals(1, state.entries.size)
        assertEquals("Google", state.entries.first().title)
    }

    @Test
    fun `should call delete use case when onDeleteEntry is called`() = runTest {
        // Given
        val entry = PasswordEntry(id = "1", title = "Test")
        coEvery { deletePasswordEntry(entry) } returns Result.success(Unit)

        viewModel = PasswordListViewModel(getAllPasswords, deletePasswordEntry, toggleFavoriteEntry)

        // When
        viewModel.onDeleteEntry(entry)

        // Then
        coVerify { deletePasswordEntry(entry) }
    }

    @Test
    fun `should update state error when delete fails`() = runTest {
        // Given
        val entry = PasswordEntry(id = "1")
        val errorMessage = "Database error"
        coEvery { deletePasswordEntry(entry) } returns Result.failure(Exception(errorMessage))

        viewModel = PasswordListViewModel(getAllPasswords, deletePasswordEntry, toggleFavoriteEntry)

        // When
        viewModel.onDeleteEntry(entry)

        // Then
        val state = viewModel.state.first()
        assertTrue(state.error?.contains(errorMessage) == true)
    }
}
```

### Reglas de Test

```kotlin
@get:Rule
val coroutinesTestRule = CoroutinesTestRule()

@get:Rule
val mockkRule = MockkRule()
```

---

## Mejores Prácticas

### 1. Estado Inmutable Público

```kotlin
// ✅ CORRECTO
private val _state = MutableStateFlow(State())
val state: StateFlow<State> = _state.asStateFlow()

// ❌ INCORRECTO
val state = MutableStateFlow(State())  // Expose mutable
```

### 2. Usar viewModelScope

```kotlin
// ✅ CORRECTO
fun onLoad() {
    viewModelScope.launch {
        useCase()
    }
}

// ❌ INCORRECTO
fun onLoad() {
    CoroutineScope(Dispatchers.Default).launch {  // No vinculado al ciclo de vida
        useCase()
    }
}
```

### 3. Copiar Estado Correctamente

```kotlin
// ✅ CORRECTO
_state.value = _state.value.copy(
    entries = newEntries,
    isLoading = false
)

// ❌ INCORRECTO
_state.value = State(
    entries = newEntries,
    isLoading = false,
    // ... recrear todo el estado es propenso a errores
)
```

### 4. Manejar Errores Consistentemente

```kotlin
// ✅ CORRECTO
useCase()
    .onSuccess { /* ... */ }
    .onFailure { error ->
        _state.value = _state.value.copy(error = error.message)
    }

// ❌ INCORRECTO
try {
    useCase()
} catch (e: Exception) {
    // Sin manejo de error
}
```

### 5. Limpiar Recursos en onCleared

```kotlin
override fun onCleared() {
    super.onCleared()
    // Cancelar jobs si las hay
    // Cerrar flows si es necesario
}
```

---

**Documentación Relacionada:**
- [States](states.md)
- [Use Cases](../domain/usecases.md)
- [Testing Guide](../testing-guide.md)
