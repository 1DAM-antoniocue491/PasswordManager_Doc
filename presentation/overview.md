# Capa de Presentación

## Visión General

La capa de presentación es responsable de:
- Mostrar la interfaz de usuario con Jetpack Compose
- Capturar interacciones del usuario
- Gestionar estados de UI
- Navegación entre pantallas
- Validación de entrada de usuario

## Arquitectura MVVM

```
┌─────────────────────────────────────────────────────────────────┐
│                         VIEW (UI)                                │
│  @Composable screens que observan StateFlow<State>               │
│  Emite eventos → ViewModel                                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       VIEWMODEL                                  │
│  Expone StateFlow<State>                                         │
│  Procesa eventos de UI                                           │
│  Coordina con Use Cases                                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       USE CASES                                  │
│  Lógica de negocio específica                                    │
└─────────────────────────────────────────────────────────────────┘
```

## Estructura de Directorios

```
presentation/
├── ui/
│   ├── screens/                # Pantallas completas
│   │   ├── LoginScreen.kt
│   │   ├── HomeScreen.kt
│   │   ├── PasswordListScreen.kt
│   │   ├── PasswordDetailScreen.kt
│   │   ├── PasswordFormScreen.kt
│   │   ├── PasswordGeneratorScreen.kt
│   │   ├── CategoryManagementScreen.kt
│   │   ├── SettingsScreen.kt
│   │   ├── BackupScreen.kt
│   │   ├── AuditScreen.kt
│   │   ├── StatisticsScreen.kt
│   │   ├── OnboardingScreen.kt
│   │   └── ChangePasswordScreen.kt
│   │
│   ├── components/             # Componentes reutilizables
│   │   ├── SearchBar.kt
│   │   ├── CategoryFilterChips.kt
│   │   ├── CategoryPicker.kt
│   │   ├── PasswordGeneratorWidget.kt
│   │   └── ColorPickerDialog.kt
│   │
│   └── state/                  # Estados de UI
│       ├── PasswordListState.kt
│       ├── PasswordFormState.kt
│       └── PasswordDetailState.kt
│
├── viewmodel/                  # ViewModels
│   ├── AuthViewModel.kt
│   ├── PasswordListViewModel.kt
│   ├── PasswordFormViewModel.kt
│   ├── PasswordDetailViewModel.kt
│   ├── PasswordGeneratorViewModel.kt
│   ├── CategoryManagementViewModel.kt
│   ├── SettingsViewModel.kt
│   ├── BackupViewModel.kt
│   ├── AuditViewModel.kt
│   ├── StatisticsViewModel.kt
│   └── ChangePasswordViewModel.kt
│
├── navigation/
│   └── NavGraph.kt             # Grafo de navegación
│
├── theme/                      # Tema Material 3
│   ├── Color.kt
│   ├── Theme.kt
│   └── Type.kt
│
├── widget/                     # Widgets
│   └── PasswordGeneratorWidget.kt
│
├── MainActivity.kt             # Actividad principal
└── AutoLockManager.kt          # Gestión de bloqueo automático
```

## Pantallas (Screens)

### LoginScreen

Pantalla de autenticación inicial.

**Archivo**: `LoginScreen.kt`

**Estado**:
```kotlin
data class LoginState(
    val password: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
    val isBiometricAvailable: Boolean = false
)
```

**Eventos**:
- `onPasswordChange(String)` - Usuario escribe password
- `onLoginClick()` - Iniciar sesión
- `onBiometricClick()` - Autenticar con biometría

**Flujo**:
```
Usuario ingresa password → onPasswordChange → Actualiza state
Usuario pulsa "Entrar" → onLoginClick → ViewModel.authenticate()
                                            ↓
                                    Éxito → Navegar a Home
                                    Error → Mostrar error
```

### HomeScreen

Pantalla principal de inicio con menú de navegación.

**Archivo**: `HomeScreen.kt`

**Elementos**:
- Header con título de la app
- Grid de opciones de navegación
- Barra de estado con información de sesión

**Opciones de navegación**:
| Opción | Icono | Destino |
|--------|-------|---------|
| Contraseñas | FaLock | PasswordListScreen |
| Generador | FaKey | PasswordGeneratorScreen |
| Categorías | FaFolders | CategoryManagementScreen |
| Copia de Seguridad | FaDownload | BackupScreen |
| Auditoría | FaShield | AuditScreen |
| Estadísticas | FaChartBar | StatisticsScreen |
| Configuración | FaCog | SettingsScreen |

### PasswordListScreen

Lista de contraseñas con búsqueda y filtrado.

**Archivo**: `PasswordListScreen.kt`

**Diseño**:
- SearchBar en la parte superior
- Filtro de categorías (chips horizontales)
- Lista dividida en dos secciones:
  - **Favoritos**: Contraseñas marcadas como favoritas
  - **Otras contraseñas**: Resto de contraseñas

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

**Características**:
- Búsqueda en tiempo real (título, username, URL)
- Filtrado por categoría
- Sección de favoritos primero
- Long press para eliminar
- Botón favorito en cada entrada

### PasswordDetailScreen

Muestra el detalle de una contraseña específica.

**Archivo**: `PasswordDetailScreen.kt`

**Información mostrada**:
- Título
- Username (con botón de copiar)
- Password (oculta/visible, con botón de copiar)
- URL (clicable)
- Notas
- Categoría (con chip de color)
- Fecha de creación/actualización

**Acciones**:
- Editar → Navega a PasswordFormScreen
- Eliminar → Muestra diálogo de confirmación
- Copiar username → Copia al portapapeles
- Copiar password → Copia al portapapeles

### PasswordFormScreen

Formulario para crear o editar contraseñas.

**Archivo**: `PasswordFormScreen.kt`

**Campos**:
| Campo | Tipo | Requerido |
|-------|------|-----------|
| Título | TextField | Sí |
| Username | TextField | Sí |
| Password | TextField + Generator | Sí |
| URL | TextField | No |
| Notas | TextField | No |
| Categoría | Selector | Sí |
| Favorito | Checkbox | No |

**Validaciones**:
- Título no vacío
- Username no vacío
- Password no vacío
- URL válida (si se proporciona)

**Generador integrado**:
- Botón "Generar" abre widget de generación
- Opciones: longitud, tipos de caracteres
- Vista previa en tiempo real

### PasswordGeneratorScreen

Generador de contraseñas seguras.

**Archivo**: `PasswordGeneratorScreen.kt`

**Opciones**:
- Longitud (slider 8-128)
- Incluir mayúsculas (checkbox)
- Incluir minúsculas (checkbox)
- Incluir números (checkbox)
- Incluir símbolos (checkbox)

**Características**:
- Vista previa en tiempo real
- Botón de copiar al portapapeles
- Medidor de fortaleza (débil/medio/fuerte)
- Generación con un toque

### CategoryManagementScreen

Gestión de categorías personalizadas.

**Archivo**: `CategoryManagementScreen.kt`

**Funcionalidades**:
- Lista de categorías (scroll vertical)
- Botón "Nueva categoría"
- Editar categoría existente (nombre, color, icono)
- Eliminar categoría (solo personalizadas)

**Diálogo de creación/edición**:
- Nombre (TextField)
- Selector de color (paleta predefinida)
- Selector de icono (Font Awesome)

### SettingsScreen

Configuración de la aplicación.

**Archivo**: `SettingsScreen.kt`

**Opciones**:

**Apariencia**:
- Tema (Claro/Oscuro/Automático)

**Seguridad**:
- Autenticación biométrica (toggle)
- Timeout de bloqueo automático (selector)
- Cambiar contraseña maestra

**Datos**:
- Exportar backup
- Importar desde CSV
- Borrar todos los datos

### BackupScreen

Gestión de copias de seguridad.

**Archivo**: `BackupScreen.kt`

**Exportar**:
- Seleccionar ubicación de archivo
- Cifrado con clave maestra
- Formato: JSON cifrado

**Importar**:
- Seleccionar archivo de backup
- Validar integridad
- Fusionar o reemplazar datos

### AuditScreen

Auditoría de seguridad de contraseñas.

**Archivo**: `AuditScreen.kt`

**Métricas**:
- Total de contraseñas
- Contraseñas débiles
- Contraseñas reutilizadas
- Fortaleza promedio

**Lista de problemas**:
- Contraseñas < 8 caracteres
- Sin mayúsculas/números/símbolos
- Patrones comunes detectados

### StatisticsScreen

Estadísticas de uso y seguridad.

**Archivo**: `StatisticsScreen.kt`

**Estadísticas**:
- Gráfico de contraseñas por categoría
- Evolución temporal (si hay histórico)
- Porcentaje de fortaleza
- Contraseñas por tipo de servicio

### OnboardingScreen

Pantalla de bienvenida para primer uso.

**Archivo**: `OnboardingScreen.kt`

**Contenido**:
- Bienvenida a la aplicación
- Explicación de características
- Configuración de contraseña maestra
- Configuración de biometría (opcional)

### ChangePasswordScreen

Cambio de contraseña maestra.

**Archivo**: `ChangePasswordScreen.kt`

**Campos**:
- Contraseña actual
- Nueva contraseña
- Confirmar nueva contraseña

**Proceso**:
1. Verificar contraseña actual
2. Validar nueva contraseña (fortaleza)
3. Confirmar coincidencia
4. Re-cifrar todas las contraseñas con nueva clave

## Componentes Reutilizables

### SearchBar

Barra de búsqueda con icono y texto de ayuda.

```kotlin
@Composable
fun SearchBar(
    query: String,
    onQueryChanged: (String) -> Unit,
    placeholder: String,
    modifier: Modifier = Modifier
) {
    OutlinedTextField(
        value = query,
        onValueChange = onQueryChanged,
        placeholder = { Text(placeholder) },
        leadingIcon = {
            Icon(Icons.Default.Search, contentDescription = null)
        },
        trailingIcon = {
            if (query.isNotEmpty()) {
                IconButton(onClick = { onQueryChanged("") }) {
                    Icon(Icons.Default.Clear, contentDescription = "Limpiar")
                }
            }
        },
        singleLine = true,
        modifier = modifier.fillMaxWidth()
    )
}
```

### CategoryFilterChips

Chips horizontales para filtrar por categoría.

```kotlin
@Composable
fun CategoryFilterChips(
    categories: List<Category>,
    selectedCategoryId: String?,
    onCategorySelected: (String?) -> Unit,
    onResetAllFilters: () -> Unit,
    modifier: Modifier = Modifier
) {
    Row(modifier = modifier.horizontalScroll(rememberScrollState())) {
        // Chip "Todas"
        FilterChip(
            selected = selectedCategoryId == null,
            onClick = { onResetAllFilters() },
            label = { Text("Todas") }
        )
        
        Spacer(modifier = Modifier.width(8.dp))
        
        // Chips de categorías
        categories.forEach { category ->
            FilterChip(
                selected = selectedCategoryId == category.id,
                onClick = { onCategorySelected(category.id) },
                label = { Text(category.name) },
                colors = FilterChipDefaults.filterChipColors(
                    selectedContainerColor = Color(category.color)
                )
            )
            Spacer(modifier = Modifier.width(8.dp))
        }
    }
}
```

### CategoryPicker

Selector de categorías para formularios.

```kotlin
@Composable
fun CategoryPicker(
    categories: List<Category>,
    selectedCategory: Category?,
    onCategorySelected: (Category) -> Unit,
    modifier: Modifier = Modifier
) {
    var expanded by remember { mutableStateOf(false) }
    
    ExposedDropdownMenuBox(
        expanded = expanded,
        onExpandedChange = { expanded = it }
    ) {
        OutlinedTextField(
            value = selectedCategory?.name ?: "",
            onValueChange = {},
            readOnly = true,
            label = { Text("Categoría") },
            trailingIcon = { ExposedDropdownMenuDefaults.TrailingIcon(expanded = expanded) },
            modifier = modifier.menuAnchor()
        )
        
        ExposedDropdownMenu(
            expanded = expanded,
            onDismissRequest = { expanded = false }
        ) {
            categories.forEach { category ->
                DropdownMenuItem(
                    text = { 
                        Row(verticalAlignment = Alignment.CenterVertically) {
                            Box(
                                modifier = Modifier
                                    .size(16.dp)
                                    .background(Color(category.color), CircleShape)
                            )
                            Spacer(modifier = Modifier.width(8.dp))
                            Text(category.name)
                        }
                    },
                    onClick = {
                        onCategorySelected(category)
                        expanded = false
                    }
                )
            }
        }
    }
}
```

### PasswordGeneratorWidget

Widget para generar contraseñas dentro de formularios.

```kotlin
@Composable
fun PasswordGeneratorWidget(
    onGenerate: (PasswordOptions) -> String,
    onPasswordGenerated: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    var showOptions by remember { mutableStateOf(false) }
    var options by remember { mutableStateOf(PasswordOptions()) }
    var generatedPassword by remember { mutableStateOf("") }
    
    Column(modifier = modifier) {
        Row(
            horizontalArrangement = Arrangement.spacedBy(8.dp),
            modifier = Modifier.fillMaxWidth()
        ) {
            Button(
                onClick = { showOptions = true },
                modifier = Modifier.weight(1f)
            ) {
                Icon(Icons.Default.Settings, contentDescription = null)
                Spacer(modifier = Modifier.width(8.dp))
                Text("Opciones")
            }
            
            Button(
                onClick = {
                    generatedPassword = onGenerate(options)
                    onPasswordGenerated(generatedPassword)
                },
                modifier = Modifier.weight(1f)
            ) {
                Icon(Icons.Default.Refresh, contentDescription = null)
                Spacer(modifier = Modifier.width(8.dp))
                Text("Generar")
            }
        }
        
        if (generatedPassword.isNotEmpty()) {
            Spacer(modifier = Modifier.height(8.dp))
            OutlinedTextField(
                value = generatedPassword,
                onValueChange = {},
                readOnly = true,
                label = { Text("Contraseña generada") },
                trailingIcon = {
                    IconButton(onClick = { /* Copiar al portapapeles */ }) {
                        Icon(Icons.Default.ContentCopy, contentDescription = "Copiar")
                    }
                },
                modifier = Modifier.fillMaxWidth()
            )
        }
        
        if (showOptions) {
            PasswordOptionsDialog(
                options = options,
                onOptionsChange = { options = it },
                onDismiss = { showOptions = false }
            )
        }
    }
}
```

### ColorPickerDialog

Diálogo para seleccionar color de categoría.

```kotlin
@Composable
fun ColorPickerDialog(
    selectedColor: Int,
    onColorSelected: (Int) -> Unit,
    onDismiss: () -> Unit
) {
    val colors = listOf(
        0xFFFF5722, 0xFFE91E63, 0xFF9C27B0, 0xFF673AB7,
        0xFF3F51B5, 0xFF2196F3, 0xFF03A9F4, 0xFF00BCD4,
        0xFF009688, 0xFF4CAF50, 0xFF8BC34A, 0xFFCDDC39,
        0xFFFFEB3B, 0xFFFFC107, 0xFFFF9800, 0xFFFF5722
    )
    
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Seleccionar color") },
        text = {
            LazyVerticalGrid(columns = GridCells.Fixed(4)) {
                items(colors) { color ->
                    IconButton(onClick = {
                        onColorSelected(color)
                        onDismiss()
                    }) {
                        Box(
                            modifier = Modifier
                                .size(48.dp)
                                .background(Color(color), CircleShape)
                        ) {
                            if (color == selectedColor) {
                                Icon(
                                    Icons.Default.Check,
                                    contentDescription = null,
                                    tint = Color.White,
                                    modifier = Modifier.align(Alignment.Center)
                                )
                            }
                        }
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

## ViewModels

### Patrón Base

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

### PasswordListViewModel

Gestiona la lista de contraseñas.

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

### PasswordFormViewModel

Gestiona el formulario de creación/edición.

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
            // Validar campos
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

### AuthViewModel

Gestiona autenticación del usuario.

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
        // La biometría desbloquea la clave del Keystore
        // que luego se usa para verificar el password
        viewModelScope.launch {
            // Completar flujo de autenticación
        }
    }
}
```

## Estados de UI (State)

### PasswordListState

```kotlin
data class PasswordListState(
    val entries: List<PasswordEntry> = emptyList(),
    val searchQuery: String = "",
    val selectedCategoryId: String? = null,
    val isLoading: Boolean = true,
    val error: String? = null
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
    val selectedCategory: Category? = null,
    val isFavorite: Boolean = false,
    val error: String? = null,
    val isSaving: Boolean = false
)

enum class FormField {
    TITLE, USERNAME, PASSWORD, URL, NOTES
}
```

### PasswordDetailState

```kotlin
data class PasswordDetailState(
    val entry: PasswordEntry? = null,
    val category: Category? = null,
    val isLoading: Boolean = true,
    val showPassword: Boolean = false,
    val error: String? = null
)
```

## Navegación

### NavGraph

El grafo de navegación define todas las rutas:

```kotlin
sealed class Screen(val route: String) {
    object Login : Screen("login")
    object Home : Screen("home")
    object PasswordList : Screen("password_list")
    object PasswordDetail : Screen("password_detail/{id}")
    object PasswordForm : Screen("password_form")
    object PasswordFormWithId : Screen("password_form/{id}")
    object PasswordGenerator : Screen("password_generator")
    object CategoryManagement : Screen("category_management")
    object Settings : Screen("settings")
    object Backup : Screen("backup")
    object Audit : Screen("audit")
    object Statistics : Screen("statistics")
    object Onboarding : Screen("onboarding")
    object ChangePassword : Screen("change_password")
}

@Composable
fun NavGraph(
    navController: NavController = rememberNavController()
) {
    NavHost(
        navController = navController,
        startDestination = "login"
    ) {
        composable("login") {
            LoginScreen(
                onNavigateToHome = {
                    navController.navigate("home") {
                        popUpTo("login") { inclusive = true }
                    }
                }
            )
        }
        
        composable("home") {
            HomeScreen(
                onNavigateToPasswordList = {
                    navController.navigate("password_list")
                },
                onNavigateToGenerator = {
                    navController.navigate("password_generator")
                },
                // ... más destinos
            )
        }
        
        composable(
            route = "password_detail/{id}",
            arguments = listOf(navArgument("id") { type = NavType.StringType })
        ) { backStackEntry ->
            val id = backStackEntry.arguments?.getString("id") ?: return@composable
            PasswordDetailScreen(
                entryId = id,
                onNavigateToEdit = {
                    navController.navigate("password_form/$id")
                },
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
        
        composable("password_form") {
            PasswordFormScreen(
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
        
        // ... más rutas
    }
}
```

## Componentes UI Reutilizables

### SearchBar

**Archivo**: `components/SearchBar.kt`

Barra de búsqueda reutilizable con icono y botón de limpiar.

```kotlin
@Composable
fun SearchBar(
    query: String,
    onQueryChanged: (String) -> Unit,
    placeholder: String = "Buscar...",
    modifier: Modifier = Modifier
) {
    OutlinedTextField(
        value = query,
        onValueChange = onQueryChanged,
        placeholder = { Text(placeholder) },
        leadingIcon = {
            Icon(
                imageVector = Icons.Default.Search,
                contentDescription = null,
                tint = MaterialTheme.colorScheme.onSurfaceVariant
            )
        },
        trailingIcon = {
            if (query.isNotEmpty()) {
                IconButton(onClick = { onQueryChanged("") }) {
                    Icon(
                        imageVector = Icons.Default.Close,
                        contentDescription = "Limpiar búsqueda",
                        tint = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                }
            }
        },
        singleLine = true,
        colors = OutlinedTextFieldDefaults.colors(
            focusedBorderColor = MaterialTheme.colorScheme.primary,
            unfocusedBorderColor = MaterialTheme.colorScheme.outline
        ),
        modifier = modifier.fillMaxWidth()
    )
}
```

**Características**:
- Icono de búsqueda a la izquierda
- Botón de limpiar (X) que aparece solo cuando hay texto
- Bordes dinámicos con el tema (primary cuando enfocado)
- Una sola línea de texto
- Ocupa todo el ancho disponible

**Uso**:
```kotlin
var searchQuery by remember { mutableStateOf("") }

SearchBar(
    query = searchQuery,
    onQueryChanged = { searchQuery = it },
    placeholder = "Buscar contraseñas..."
)
```

---

### CategoryPicker

**Archivo**: `components/CategoryPicker.kt`

Selector de categorías en formato grid con soporte para crear nuevas.

```kotlin
@Composable
fun CategoryPicker(
    categories: List<Category>,
    selectedCategory: Category?,
    onCategorySelected: (Category) -> Unit,
    onDismissRequest: () -> Unit,
    onCreateCategory: ((String, Int, String) -> Unit)? = null,
    modifier: Modifier = Modifier
) {
    var showNewCategoryDialog by remember { mutableStateOf(false) }
    
    AlertDialog(
        onDismissRequest = onDismissRequest,
        title = { Text("Seleccionar categoría") },
        text = {
            LazyVerticalGrid(columns = GridCells.Fixed(3)) {
                items(categories.size) { index ->
                    CategoryItem(
                        category = categories[index],
                        isSelected = selectedCategory?.id == categories[index].id,
                        onClick = { onCategorySelected(categories[index]) }
                    )
                }
                if (onCreateCategory != null) {
                    item { AddCategoryItem(onClick = { showNewCategoryDialog = true }) }
                }
            }
        },
        confirmButton = {
            TextButton(onClick = onDismissRequest) { Text("Cancelar") }
        }
    )
}
```

**Características**:
- Grid de 3 columnas
- Cada categoría muestra color + icono + nombre
- Indicador visual de selección (borde primary)
- Botón "Nueva" para añadir categoría personalizada
- Dialog secundario para crear nueva categoría con selector de color

**Estructura del Item**:
```kotlin
@Composable
private fun CategoryItem(
    category: Category,
    isSelected: Boolean,
    onClick: () -> Unit
) {
    Column(horizontalAlignment = Alignment.CenterHorizontally) {
        // Círculo de color con icono
        Box(
            modifier = Modifier
                .size(48.dp)
                .background(Color(category.color), CircleShape)
                .then(if (isSelected) Modifier.padding(2.dp)
                    .background(MaterialTheme.colorScheme.primary, CircleShape) 
                    else Modifier)
        ) {
            Icon(imageVector = Icons.Default.Star, tint = Color.White)
        }
        Text(text = category.name, style = MaterialTheme.typography.labelSmall)
    }
}
```

---

### CategoryDropdown

**Archivo**: `components/CategoryPicker.kt`

Versión simplificada como dropdown para formularios.

```kotlin
@Composable
fun CategoryDropdown(
    categories: List<Category>,
    selectedCategory: Category?,
    onCategorySelected: (Category) -> Unit,
    modifier: Modifier = Modifier
) {
    var expanded by remember { mutableStateOf(false) }
    
    ExposedDropdownMenuBox(
        expanded = expanded,
        onExpandedChange = { expanded = !expanded }
    ) {
        OutlinedTextField(
            value = selectedCategory?.name ?: "",
            onValueChange = {},
            readOnly = true,
            label = { Text("Categoría") },
            trailingIcon = { ExposedDropdownMenuDefaults.TrailingIcon(expanded) },
            modifier = Modifier.menuAnchor().fillMaxWidth()
        )
        
        ExposedDropdownMenu(expanded, { expanded = false }) {
            categories.forEach { category ->
                DropdownMenuItem(
                    text = {
                        Row {
                            Box(modifier = Modifier.size(16.dp)
                                .background(Color(category.color), CircleShape))
                            Text(category.name)
                        }
                    },
                    onClick = {
                        onCategorySelected(category)
                        expanded = false
                    }
                )
            }
        }
    }
}
```

**Uso típico**:
```kotlin
CategoryDropdown(
    categories = categories,
    selectedCategory = selectedCategory,
    onCategorySelected = { category ->
        viewModel.onCategorySelected(category)
    }
)
```

---

### ColorPickerDialog

**Archivo**: `components/ColorPickerDialog.kt`

Dialog para seleccionar color de una paleta predefinida.

```kotlin
@Composable
fun ColorPickerDialog(
    currentColor: Int,
    onColorSelected: (Int) -> Unit,
    onDismissRequest: () -> Unit
) {
    var selectedColor by remember { mutableStateOf(currentColor) }
    
    AlertDialog(
        onDismissRequest = onDismissRequest,
        title = { Text("Seleccionar color") },
        text = {
            Column {
                // Vista previa
                Row {
                    Text("Color seleccionado:")
                    Box(modifier = Modifier
                        .size(24.dp)
                        .clip(CircleShape)
                        .background(Color(selectedColor)))
                    Text("#${selectedColor.toString(16).uppercase()}")
                }
                
                // Grid de colores (6 columnas)
                LazyVerticalGrid(
                    columns = GridCells.Fixed(6),
                    modifier = Modifier.height(200.dp)
                ) {
                    items(paletteColors.size) { index ->
                        Box(modifier = Modifier
                            .size(40.dp)
                            .clip(CircleShape)
                            .background(Color(paletteColors[index]))
                            .clickable { selectedColor = paletteColors[index] }
                            .then(if (selectedColor == paletteColors[index])
                                Modifier.padding(3.dp)
                                    .background(MaterialTheme.colorScheme.onSurface, CircleShape)
                                else Modifier))
                    }
                }
            }
        },
        confirmButton = {
            TextButton(onClick = { 
                onColorSelected(selectedColor)
                onDismissRequest()
            }) { Text("Aceptar") }
        },
        dismissButton = {
            TextButton(onClick = onDismissRequest) { Text("Cancelar") }
        }
    )
}
```

**Paleta de Colores** (28 colores):
| Grupo | Colores (HEX) |
|-------|---------------|
| Rojos | #E74C3C, #C0392B, #EA4335 |
| Naranjas | #F39C12, #E67E22, #D35400 |
| Amarillos | #F1C40F, #FECA57, #FF9F43 |
| Verdes | #27AE60, #2ECC71, #1ABC9C |
| Azules | #3498DB, #2980B9, #4267B2, #5DADE2 |
| Morados | #8E44AD, #9B59B6, #A569BD |
| Rosas | #E91E63, #FF4081, #FF69B4 |
| Grises | #95A5A6, #7F8C8D, #BDC3C7, #34495E |
| Extra | #000000, #FFFFFF |

---

## Tema (Material 3)

### Color.kt

Documentación completa de colores y esquema de la aplicación.

**Archivo**: `Color.kt`

#### Colores del Tema Material 3

```kotlin
// Tema Oscuro (Dark Theme)
val Purple80 = Color(0xFFD0BCFF)      // Primary Dark
val PurpleGrey80 = Color(0xFFCCC2DC)  // Secondary Dark
val Pink80 = Color(0xFFEFB8C8)        // Tertiary Dark

// Tema Claro (Light Theme)
val Purple40 = Color(0xFF6650a4)      // Primary Light
val PurpleGrey40 = Color(0xFF625b71)  // Secondary Light
val Pink40 = Color(0xFF7D5260)        // Tertiary Light

// Colores de estado
val Red40 = Color(0xFFB3261E)         // Error
val Red80 = Color(0xFFF2B8B5)         // Error Dark
```

#### Esquema de Colores Completo

```kotlin
private val DarkColorScheme = darkColorScheme(
    primary = Purple80,
    secondary = PurpleGrey80,
    tertiary = Pink80,
    error = Red80,
    background = Color(0xFF1C1B1F),
    surface = Color(0xFF1C1B1F),
    onPrimary = Color(0xFF381E72),
    onSecondary = Color(0xFF332D41),
    onTertiary = Color(0xFF3D111B),
    onError = Color(0xFF690005),
    onBackground = Color(0xFFE6E1E5),
    onSurface = Color(0xFFE6E1E5)
)

private val LightColorScheme = lightColorScheme(
    primary = Purple40,
    secondary = PurpleGrey40,
    tertiary = Pink40,
    error = Red40,
    background = Color(0xFFFFFBFE),
    surface = Color(0xFFFFFBFE),
    onPrimary = Color(0xFFFFFFFF),
    onSecondary = Color(0xFFFFFFFF),
    onTertiary = Color(0xFFFFFFFF),
    onError = Color(0xFFFFFFFF),
    onBackground = Color(0xFF1C1B1F),
    onSurface = Color(0xFF1C1B1F)
)
```

### Theme.kt

**Archivo**: `Theme.kt`

```kotlin
@Composable
fun PasswordManagerTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

**Características**:
- `darkTheme`: Detecta automáticamente el tema del sistema
- `dynamicColor`: Usa colores dinámicos de Material You (Android 12+)
- Compatible con Android 8.0 (API 26) en adelante

### Type.kt

**Archivo**: `Type.kt`

```kotlin
val Typography = Typography(
    displayLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp,
        letterSpacing = (-0.25).sp
    ),
    displayMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 45.sp,
        lineHeight = 52.sp,
        letterSpacing = 0.sp
    ),
    displaySmall = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 36.sp,
        lineHeight = 44.sp,
        letterSpacing = 0.sp
    ),
    headlineLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 32.sp,
        lineHeight = 40.sp,
        letterSpacing = 0.sp
    ),
    headlineMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 28.sp,
        lineHeight = 36.sp,
        letterSpacing = 0.sp
    ),
    headlineSmall = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 24.sp,
        lineHeight = 32.sp,
        letterSpacing = 0.sp
    ),
    titleLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 22.sp,
        lineHeight = 28.sp,
        letterSpacing = 0.sp
    ),
    titleMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.15.sp
    ),
    titleSmall = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.1.sp
    ),
    bodyLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    ),
    bodyMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.25.sp
    ),
    bodySmall = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 12.sp,
        lineHeight = 16.sp,
        letterSpacing = 0.4.sp
    ),
    labelLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.1.sp
    ),
    labelMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 12.sp,
        lineHeight = 16.sp,
        letterSpacing = 0.5.sp
    ),
    labelSmall = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 11.sp,
        lineHeight = 16.sp,
        letterSpacing = 0.5.sp
    )
)
```

---

## ViewModels Adicionales

### PasswordDetailViewModel

**Archivo**: `viewmodel/PasswordDetailViewModel.kt`

Gestiona la pantalla de detalle de una contraseña individual.

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

**Funcionalidades**:
- Carga entrada por ID
- Alterna visibilidad de contraseña (icono ojo)
- Copia username al portapapeles
- Copia password al portapapeles
- Manejo de errores

---

### PasswordGeneratorViewModel

**Archivo**: `viewmodel/PasswordGeneratorViewModel.kt`

Gestiona la generación de contraseñas seguras.

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
            // ... más eventos
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

**Eventos**:
```kotlin
sealed class PasswordGeneratorEvent {
    data class OnLengthChanged(val length: Int) : PasswordGeneratorEvent()
    data class OnUseUppercaseChanged(val use: Boolean) : PasswordGeneratorEvent()
    data class OnUseLowercaseChanged(val use: Boolean) : PasswordGeneratorEvent()
    data class OnUseDigitsChanged(val use: Boolean) : PasswordGeneratorEvent()
    data class OnUseSymbolsChanged(val use: Boolean) : PasswordGeneratorEvent()
    data class OnExcludeAmbiguousChanged(val exclude: Boolean) : PasswordGeneratorEvent()
    data object OnGenerateClicked : PasswordGeneratorEvent()
    data object OnCopyClicked : PasswordGeneratorEvent()
    data object OnDismissCopiedMessage : PasswordGeneratorEvent()
}
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

fun getStrengthLabel(score: Int): String = when {
    score < 40 -> "Débil"
    score < 70 -> "Media"
    score < 90 -> "Fuerte"
    else -> "Muy Fuerte"
}

fun getStrengthColor(score: Int): Long = when {
    score < 40 -> 0xFFFF0000L  // Rojo
    score < 70 -> 0xFFFFA500L  // Naranja
    score < 90 -> 0xFFFFFF00L  // Amarillo
    else -> 0xFF00FF00L        // Verde
}
```

---

### SettingsViewModel

**Archivo**: `viewmodel/SettingsViewModel.kt`

Gestiona la configuración de la aplicación.

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

**Eventos**:
```kotlin
sealed class SettingsEvent {
    data object ShowTimeoutPicker : SettingsEvent()
    data object DismissTimeoutPicker : SettingsEvent()
    data object ShowThemePicker : SettingsEvent()
    data object DismissThemePicker : SettingsEvent()
    data class OnLockTimeoutChanged(val minutes: Int) : SettingsEvent()
    data class OnBiometricToggled(val enabled: Boolean) : SettingsEvent()
    data class OnThemeModeChanged(val mode: ThemeMode) : SettingsEvent()
    data object DismissMessage : SettingsEvent()
}
```

---

### AuditViewModel

**Archivo**: `viewmodel/AuditViewModel.kt`

Gestiona la auditoría de contraseñas débiles.

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

### StatisticsViewModel

**Archivo**: `viewmodel/StatisticsViewModel.kt`

Gestiona las estadísticas de seguridad.

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

### BackupViewModel

**Archivo**: `viewmodel/BackupViewModel.kt`

Gestiona exportación e importación de backups.

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
            
            // Verificar sesión activa
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

### ChangePasswordViewModel

**Archivo**: `viewmodel/ChangePasswordViewModel.kt`

Gestiona el cambio de contraseña maestra.

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
            if (newPassword.contentEquals(confirmPassword)) {
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
                    // Limpiar sesión y forzar re-login
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

---

## Navegación Detallada

### Grafo Completo de Navegación

**Archivo**: `NavGraph.kt`

```kotlin
sealed class Screen(val route: String) {
    object Login : Screen("login")
    object Onboarding : Screen("onboarding")
    object Home : Screen("home")
    object PasswordList : Screen("password_list")
    object PasswordDetail : Screen("password_detail/{id}")
    object PasswordForm : Screen("password_form")
    object PasswordFormEdit : Screen("password_form/{id}")
    object PasswordGenerator : Screen("password_generator")
    object CategoryManagement : Screen("category_management")
    object Settings : Screen("settings")
    object Backup : Screen("backup")
    object Audit : Screen("audit")
    object Statistics : Screen("statistics")
    object ChangePassword : Screen("change_password")
}
```

### Flujo de Navegación Principal

```
┌─────────────────────────────────────────────────────────────┐
│                    ¿Primera vez?                             │
│                          │                                   │
│              ┌───────────┴───────────┐                       │
│              ▼                       ▼                       │
│         Onboarding               Login                       │
│              │                       │                       │
│              └───────────┬───────────┘                       │
│                          ▼                                   │
│                      Home Screen                             │
│                          │                                   │
│        ┌─────────────────┼─────────────────┐                 │
│        ▼                 ▼                 ▼                 │
│  PasswordList     PasswordGenerator   CategoryMgmt           │
│       │                                      │               │
│       ▼                                      ▼               │
│  PasswordDetail                         (editar)              │
│       │                                                      │
│       ▼                                                      │
│  PasswordForm (edit)                                         │
│                                                              │
│  Otras rutas desde Home:                                     │
│  - Settings → ChangePassword                                 │
│  - Backup                                                    │
│  - Audit                                                     │
│  - Statistics                                                │
└─────────────────────────────────────────────────────────────┘
```

### Configuración de Rutas

```kotlin
@Composable
fun NavGraph(
    navController: NavController = rememberNavController()
) {
    NavHost(
        navController = navController,
        startDestination = determineStartDestination(),
        modifier = Modifier.padding(innerPadding)
    ) {
        // Onboarding - Solo primera vez
        composable("onboarding") {
            OnboardingScreen(
                onComplete = {
                    navController.navigate("home") {
                        popUpTo(0) { inclusive = true }
                    }
                }
            )
        }
        
        // Login
        composable("login") {
            LoginScreen(
                onLoginSuccess = {
                    navController.navigate("home") {
                        popUpTo("login") { inclusive = true }
                    }
                },
                onNavigateToOnboarding = {
                    navController.navigate("onboarding")
                }
            )
        }
        
        // Home - Pantalla principal
        composable("home") {
            HomeScreen(
                onNavigateToPasswordList = {
                    navController.navigate("password_list")
                },
                onNavigateToGenerator = {
                    navController.navigate("password_generator")
                },
                onNavigateToCategoryManagement = {
                    navController.navigate("category_management")
                },
                onNavigateToBackup = {
                    navController.navigate("backup")
                },
                onNavigateToAudit = {
                    navController.navigate("audit")
                },
                onNavigateToStatistics = {
                    navController.navigate("statistics")
                },
                onNavigateToSettings = {
                    navController.navigate("settings")
                }
            )
        }
        
        // Lista de contraseñas
        composable("password_list") {
            PasswordListScreen(
                onNavigateToDetail = { id ->
                    navController.navigate("password_detail/$id")
                },
                onNavigateToCreate = {
                    navController.navigate("password_form")
                },
                onNavigateToEdit = { id ->
                    navController.navigate("password_form/$id")
                }
            )
        }
        
        // Detalle de contraseña
        composable(
            route = "password_detail/{id}",
            arguments = listOf(navArgument("id") { type = NavType.StringType })
        ) { backStackEntry ->
            val id = backStackEntry.arguments?.getString("id") ?: return@composable
            PasswordDetailScreen(
                entryId = id,
                onNavigateToEdit = {
                    navController.navigate("password_form/$id")
                },
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
        
        // Formulario (crear)
        composable("password_form") {
            PasswordFormScreen(
                entryId = null,
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
        
        // Formulario (editar)
        composable(
            route = "password_form/{id}",
            arguments = listOf(navArgument("id") { type = NavType.StringType })
        ) { backStackEntry ->
            val id = backStackEntry.arguments?.getString("id") ?: return@composable
            PasswordFormScreen(
                entryId = id,
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
        
        // Generador de contraseñas
        composable("password_generator") {
            PasswordGeneratorScreen(
                onNavigateBack = {
                    navController.popBackStack()
                },
                onPasswordGenerated = { password ->
                    // Opcional: navegar a password_form con password
                }
            )
        }
        
        // Gestión de categorías
        composable("category_management") {
            CategoryManagementScreen(
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
        
        // Configuración
        composable("settings") {
            SettingsScreen(
                onNavigateToChangePassword = {
                    navController.navigate("change_password")
                },
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
        
        // Cambiar contraseña maestra
        composable("change_password") {
            ChangePasswordScreen(
                onNavigateBack = {
                    navController.popBackStack()
                },
                onPasswordChanged = {
                    // Forzar re-autenticación
                    navController.navigate("login") {
                        popUpTo(0) { inclusive = true }
                    }
                }
            )
        }
        
        // Backup
        composable("backup") {
            BackupScreen(
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
        
        // Auditoría
        composable("audit") {
            AuditScreen(
                onNavigateBack = {
                    navController.popBackStack()
                },
                onNavigateToPasswordDetail = { id ->
                    navController.navigate("password_detail/$id")
                }
            )
        }
        
        // Estadísticas
        composable("statistics") {
            StatisticsScreen(
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
    }
}
```

### Patrones de Navegación

#### Navegación con PopUpTo

Para evitar acumular pantallas en el backstack:

```kotlin
// Después de login exitoso
navController.navigate("home") {
    popUpTo("login") { inclusive = true }
}

// Después de completar onboarding
navController.navigate("home") {
    popUpTo(0) { inclusive = true }  // Limpia todo el backstack
}
```

#### Navegación con Argumentos

```kotlin
// Definir ruta con argumento
object PasswordDetail : Screen("password_detail/{id}")

// Navegar con argumento
navController.navigate("password_detail/$id")

// Leer argumento en destino
val id = backStackEntry.arguments?.getString("id")
```

#### Navegación Conditional

```kotlin
fun determineStartDestination(): String {
    return when {
        !hasMasterPassword() -> "onboarding"
        !isAuthenticated() -> "login"
        else -> "home"
    }
}
```

---

## MainActivity

**Archivo**: `MainActivity.kt`

```kotlin
class MainActivity : ComponentActivity() {

    private val autoLockManager: AutoLockManager by inject()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate()
        
        // Habilitar edge-to-edge
        enableEdgeToEdge()
        
        setContent {
            PasswordManagerTheme {
                Scaffold(
                    modifier = Modifier.fillMaxSize()
                ) { innerPadding ->
                    NavGraph(
                        navController = rememberNavController(),
                        innerPadding = innerPadding
                    )
                }
            }
        }
    }

    override fun onStart() {
        super.onStart()
        autoLockManager.resetTimer()
    }

    override fun onStop() {
        super.onStop()
        autoLockManager.startLockTimer()
    }
}
```

---

## AutoLockManager

**Archivo**: `AutoLockManager.kt`

Gestiona el bloqueo automático de la aplicación cuando pasa a segundo plano.

```kotlin
class AutoLockManager(
    private val settingsRepository: SettingsRepository,
    private val application: Application
) : LifecycleEventObserver {

    private var lockTimer: Job? = null
    private var lastAccessTime: Long = 0
    private var lockTimeoutMs: Long = 300_000  // 5 minutos por defecto

    fun initialize() {
        // Observar ciclo de vida de la aplicación
        ProcessLifecycleOwner.get().lifecycle.addObserver(this)
        
        // Cargar configuración de timeout
        viewModelScope.launch {
            lockTimeoutMs = settingsRepository.getLockTimeout()
        }
    }

    override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
        when (event) {
            Lifecycle.Event.ON_STOP -> {
                // App pasó a segundo plano
                lastAccessTime = System.currentTimeMillis()
                startLockTimer()
            }
            Lifecycle.Event.ON_START -> {
                // App volvió a primer plano
                stopLockTimer()
                checkIfLockRequired()
            }
            else -> {}
        }
    }

    private fun startLockTimer() {
        lockTimer = viewModelScope.launch {
            delay(lockTimeoutMs)
            // Tiempo agotado, requerir autenticación
            requireAuthentication()
        }
    }

    private fun stopLockTimer() {
        lockTimer?.cancel()
        lockTimer = null
    }

    private fun checkIfLockRequired() {
        val elapsed = System.currentTimeMillis() - lastAccessTime
        if (elapsed > lockTimeoutMs) {
            requireAuthentication()
        }
    }

    private fun requireAuthentication() {
        // Limpiar clave maestra de memoria
        authRepository.clearMasterKey()
        
        // Navegar a pantalla de login
        // (requiere referencia a NavController o usar evento)
    }

    fun resetTimer() {
        lastAccessTime = System.currentTimeMillis()
        stopLockTimer()
    }

    fun setLockTimeout(timeoutMs: Long) {
        lockTimeoutMs = timeoutMs
    }
}
```

**Configuración de Timeout**:

| Opción | Valor (ms) |
|--------|------------|
| 1 minuto | 60,000 |
| 2 minutos | 120,000 |
| 5 minutos | 300,000 (default) |
| 10 minutos | 600,000 |
| Nunca | -1 (no iniciar timer) |

---

## Widget de Generador (App Widget)

**Archivo**: `PasswordGeneratorWidget.kt` (en `presentation/widget/`)

Widget de pantalla de inicio para generar contraseñas rápidamente.

```kotlin
@Composable
fun PasswordGeneratorWidget(
    state: PasswordGeneratorState,
    onGenerate: () -> Unit,
    onCopy: (String) -> Unit
) {
    Surface(
        modifier = Modifier
            .size(180.dp)
            .padding(8.dp),
        shape = RoundedCornerShape(16.dp),
        color = MaterialTheme.colorScheme.primaryContainer
    ) {
        Column(
            modifier = Modifier.padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Icon(
                imageVector = Icons.Default.Key,
                contentDescription = null,
                modifier = Modifier.size(32.dp)
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            Text(
                text = "Generar Contraseña",
                style = MaterialTheme.typography.titleSmall
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            if (state.generatedPassword.isNotEmpty()) {
                Text(
                    text = state.generatedPassword,
                    style = MaterialTheme.typography.bodySmall,
                    maxLines = 2,
                    overflow = TextOverflow.Ellipsis
                )
            }
            
            Spacer(modifier = Modifier.weight(1f))
            
            Button(
                onClick = onGenerate,
                modifier = Modifier.fillMaxWidth()
            ) {
                Icon(
                    imageVector = Icons.Default.Refresh,
                    contentDescription = null,
                    modifier = Modifier.size(16.dp)
                )
                Spacer(modifier = Modifier.width(4.dp))
                Text("Generar")
            }
        }
    }
}
```

### Configuración del Widget

```kotlin
@Composable
fun PasswordGeneratorWidgetPreview() {
    PasswordManagerTheme {
        PasswordGeneratorWidget(
            state = PasswordGeneratorState(
                generatedPassword = "K#9mX$2pL@5nQr8Y",
                length = 16,
                includeUppercase = true,
                includeLowercase = true,
                includeNumbers = true,
                includeSymbols = true
            ),
            onGenerate = { },
            onCopy = { }
        )
    }
}
```

---

**Documentación Relacionada:**
- [Screens Overview](screens/overview.md)
- [ViewModels](viewmodels/overview.md)
- [Components](components/overview.md)

## AutoLockManager

Gestiona el bloqueo automático por inactividad.

```kotlin
class AutoLockManager(
    private val settingsRepository: SettingsRepository,
    private val application: Application
) : LifecycleEventObserver {
    
    private var lastAccessTime: Long = 0
    private var lockTimeoutMs: Long = 300_000  // 5 minutos por defecto
    private var isAppInBackground: Boolean = false
    
    fun initialize() {
        // Observar ciclo de vida de la aplicación
        ProcessLifecycleOwner.get().lifecycle.addObserver(this)
        
        // Cargar timeout de configuración
        viewModelScope.launch {
            lockTimeoutMs = settingsRepository.getLockTimeout()
        }
    }
    
    override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
        when (event) {
            Lifecycle.Event.ON_STOP -> {
                // App pasó a segundo plano
                isAppInBackground = true
                lastAccessTime = System.currentTimeMillis()
            }
            Lifecycle.Event.ON_START -> {
                // App volvió a primer plano
                isAppInBackground = false
                
                // Verificar si pasó el timeout
                val elapsed = System.currentTimeMillis() - lastAccessTime
                if (elapsed > lockTimeoutMs) {
                    // Requiere autenticación de nuevo
                    requireAuthentication()
                }
            }
            else -> {}
        }
    }
    
    fun resetTimer() {
        lastAccessTime = System.currentTimeMillis()
    }
    
    private fun requireAuthentication() {
        // Navegar a pantalla de login
        // Limpiar clave maestra de memoria
    }
}
```

## Testing de la Capa de Presentación

### UI Tests con Compose

```kotlin
@RunWith(AndroidJUnit4::class)
class PasswordListScreenTest {
    
    @get:Rule
    val composeTestRule = createComposeRule()
    
    private val fakeViewModel = FakePasswordListViewModel()
    
    @Test
    fun passwordListScreen_displaysEntries() {
        // Given
        val entries = listOf(
            PasswordEntry(id = "1", title = "Test", username = "user", 
                         password = "pass", url = null, notes = null,
                         categoryId = "cat-1", icon = null, isFavorite = false,
                         createdAt = 0, updatedAt = 0)
        )
        
        // When
        composeTestRule.setContent {
            PasswordManagerTheme {
                PasswordListScreen(
                    viewModel = fakeViewModel,
                    onNavigateToDetail = {},
                    onNavigateToCreate = {},
                    onNavigateToEdit = {},
                    categories = emptyList()
                )
            }
        }
        
        // Then
        composeTestRule
            .onNodeWithText("Test")
            .assertIsDisplayed()
    }
    
    @Test
    fun passwordListScreen_clickOnFavorite_togglesFavorite() {
        // Given
        val entry = PasswordEntry(/* ... */)
        
        // When
        composeTestRule.setContent {
            PasswordListScreen(viewModel = fakeViewModel, ...)
        }
        
        composeTestRule
            .onNodeWithContentDescription("Añadir a favoritos")
            .performClick()
        
        // Then
        verify(fakeViewModel).onToggleFavorite(entry)
    }
}
```

### ViewModel Tests

```kotlin
class PasswordListViewModelTest {
    
    private val getAllPasswords = MockkGetAllPasswords()
    private val deletePasswordEntry = MockkDeletePasswordEntry()
    private val toggleFavoriteEntry = MockkToggleFavoriteEntry()
    
    private lateinit var viewModel: PasswordListViewModel
    
    @Before
    fun setup() {
        viewModel = PasswordListViewModel(
            getAllPasswords,
            deletePasswordEntry,
            toggleFavoriteEntry
        )
    }
    
    @Test
    fun `should filter entries by search query`() = runTest {
        // Given
        val entries = listOf(
            PasswordEntry(title = "Google", /* ... */),
            PasswordEntry(title = "Facebook", /* ... */),
            PasswordEntry(title = "Amazon", /* ... */)
        )
        
        coEvery { getAllPasswords() } returns flowOf(entries)
        
        // When
        viewModel.onSearchQueryChanged("Goog")
        
        // Then
        val state = viewModel.state.first()
        assertEquals(1, state.entries.size)
        assertEquals("Google", state.entries.first().title)
    }
    
    @Test
    fun `should toggle favorite when clicked`() = runTest {
        // Given
        val entry = PasswordEntry(id = "1", /* ... */)
        coEvery { toggleFavoriteEntry(entry) } returns Result.success(Unit)
        
        // When
        viewModel.onToggleFavorite(entry)
        
        // Then
        coVerify { toggleFavoriteEntry(entry) }
    }
}
```

---

**Documentación Relacionada:**
- [Arquitectura](../arquitectura/overview.md)
- [Capa de Dominio](../domain/overview.md)
- [Navegación](navigation/overview.md)
