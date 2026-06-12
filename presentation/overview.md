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

## Tema (Material 3)

### Color.kt

```kotlin
val Purple80 = Color(0xFFD0BCFF)
val PurpleGrey80 = Color(0xFFCCC2DC)
val Pink80 = Color(0xFFEFB8C8)

val Purple40 = Color(0xFF6650a4)
val PurpleGrey40 = Color(0xFF625b71)
val Pink40 = Color(0xFF7D5260)

// Colores personalizados para la app
val PasswordManagerPrimary = Color(0xFF2196F3)
val PasswordManagerSecondary = Color(0xFF03DAC6)
val PasswordManagerError = Color(0xFFB00020)
```

### Theme.kt

```kotlin
@Composable
fun PasswordManagerTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) {
        darkColorScheme(
            primary = Purple80,
            secondary = PurpleGrey80,
            tertiary = Pink80
        )
    } else {
        lightColorScheme(
            primary = Purple40,
            secondary = PurpleGrey40,
            tertiary = Pink40
        )
    }
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

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
