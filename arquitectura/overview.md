# Arquitectura del Proyecto

## Visión General

Password Manager sigue los principios de **Clean Architecture** combinados con el patrón **MVVM** (Model-View-ViewModel) para lograr una separación clara de responsabilidades y un código mantenible y testeable.

## Capas de la Arquitectura

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                            │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌─────────────┐  │
│  │  Screens  │  │ ViewModels│  │   State   │  │ Components  │  │
│  │   (UI)    │◄─┤           │◄─┤   Flow    │  │             │  │
│  └───────────┘  └───────────┘  └───────────┘  └─────────────┘  │
│         ▲              │                                        │
│         │              ▼                                        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Navigation Graph                       │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      DOMAIN LAYER                                │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌─────────────┐  │
│  │  Models   │  │ Use Cases │  │Repositories│  │  Interfaces │  │
│  │           │  │           │  │ (Interface)│  │             │  │
│  └───────────┘  └───────────┘  └────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DATA LAYER                                 │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌─────────────┐  │
│  │  Entities │  │ Repository│  │    DAO    │  │   Security  │  │
│  │           │  │    Impl   │  │           │  │  Components │  │
│  └───────────┘  └───────────┘  └───────────┘  └─────────────┘  │
│         ▲              │              │                │        │
│         │              │              ▼                ▼        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Room Database + DataStore                   │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Capa de Presentación

### Responsabilidades
- Mostrar datos en la interfaz de usuario (Jetpack Compose)
- Capturar interacciones del usuario
- Manejar estados de UI (loading, error, éxito)
- Navegación entre pantallas

### Componentes Principales

#### Screens (Pantallas)
Composables que representan pantallas completas de la aplicación:

| Pantalla | Archivo | Descripción |
|----------|---------|-------------|
| Login | `LoginScreen.kt` | Autenticación inicial |
| Home | `HomeScreen.kt` | Pantalla principal de inicio |
| PasswordList | `PasswordListScreen.kt` | Lista de contraseñas |
| PasswordDetail | `PasswordDetailScreen.kt` | Detalle de una contraseña |
| PasswordForm | `PasswordFormScreen.kt` | Crear/editar contraseña |
| PasswordGenerator | `PasswordGeneratorScreen.kt` | Generador de contraseñas |
| CategoryManagement | `CategoryManagementScreen.kt` | Gestión de categorías |
| Settings | `SettingsScreen.kt` | Configuración de la app |
| Backup | `BackupScreen.kt` | Exportar/importar datos |
| Audit | `AuditScreen.kt` | Auditoría de seguridad |
| Statistics | `StatisticsScreen.kt` | Estadísticas de seguridad |

#### ViewModels
Gestionan el estado de la UI y coordinan con casos de uso:

```kotlin
// Ejemplo: PasswordListViewModel
class PasswordListViewModel(
    private val getAllPasswords: GetAllPasswords,
    private val deletePasswordEntry: DeletePasswordEntry,
    private val toggleFavoriteEntry: ToggleFavoriteEntry
) : ViewModel() {
    
    private val _state = MutableStateFlow(PasswordListState())
    val state: StateFlow<PasswordListState> = _state.asStateFlow()
    
    // Los ViewModels exponen StateFlow para reactividad
}
```

#### UI State
Data classes que representan el estado de cada pantalla:

```kotlin
data class PasswordListState(
    val entries: List<PasswordEntry> = emptyList(),
    val searchQuery: String = "",
    val selectedCategoryId: String? = null,
    val isLoading: Boolean = true,
    val error: String? = null
)
```

### Flujo de Datos en Presentación

```
Usuario interactúa → ViewModel → UseCase → Repository
                          ↓
                    Actualiza StateFlow
                          ↓
                    UI observa y se recompone
```

## Capa de Dominio

### Responsabilidades
- Contiene la lógica de negocio
- Define casos de uso (una acción por clase)
- Modelos de negocio independientes de Android
- Interfaces de repositorio

### Características
- **Independiente**: No depende de Android ni de frameworks
- **Pure Kotlin**: Solo código Kotlin estándar
- **Testeable**: Fácil de probar con unit tests

### Casos de Uso

#### Autenticación
| Caso de Uso | Descripción |
|-------------|-------------|
| `SetupMasterPassword` | Configura contraseña maestra inicial |
| `AuthenticateUser` | Valida credenciales del usuario |
| `ChangeMasterPassword` | Cambia la contraseña maestra |

#### Gestión de Contraseñas
| Caso de Uso | Descripción |
|-------------|-------------|
| `GetAllPasswords` | Obtiene todas las contraseñas |
| `GetPasswordById` | Obtiene una contraseña por ID |
| `CreatePasswordEntry` | Crea nueva contraseña |
| `UpdatePasswordEntry` | Actualiza contraseña existente |
| `DeletePasswordEntry` | Elimina una contraseña |
| `SearchPasswords` | Busca por título/usuario/URL |
| `FilterPasswordsByCategory` | Filtra por categoría |
| `ToggleFavoriteEntry` | Marca/desmarca como favorito |
| `GeneratePassword` | Genera contraseña segura |

#### Gestión de Categorías
| Caso de Uso | Descripción |
|-------------|-------------|
| `GetCategories` | Obtiene todas las categorías |
| `CreateCategory` | Crea nueva categoría |
| `UpdateCategory` | Actualiza categoría |
| `DeleteCategory` | Elimina categoría |
| `SeedPredefinedCategories` | Inicializa categorías por defecto |

#### Configuración
| Caso de Uso | Descripción |
|-------------|-------------|
| `GetSettings` | Obtiene configuración |
| `UpdateSettings` | Actualiza configuración |
| `GetThemeMode` | Obtiene tema (claro/oscuro) |
| `SetThemeMode` | Establece tema |
| `IsBiometricEnabled` | Verifica si biometría está activa |
| `SetBiometricEnabled` | Activa/desactiva biometría |
| `GetLockTimeout` | Obtiene tiempo de bloqueo automático |
| `SetLockTimeout` | Establece tiempo de bloqueo |

#### Copias de Seguridad
| Caso de Uso | Descripción |
|-------------|-------------|
| `ExportEncryptedBackup` | Exporta datos cifrados |
| `ImportFromCSV` | Importa desde CSV |

#### Auditoría y Estadísticas
| Caso de Uso | Descripción |
|-------------|-------------|
| `AuditWeakPasswords` | Detecta contraseñas débiles |
| `GetSecurityStatistics` | Obtiene estadísticas de seguridad |

### Modelos de Dominio

#### PasswordEntry
```kotlin
data class PasswordEntry(
    val id: String,              // UUID único
    val title: String,           // Título descriptivo
    val username: String,        // Nombre de usuario
    val password: String,        // Contraseña (descifrada)
    val url: String?,            // URL opcional
    val notes: String?,          // Notas adicionales
    val categoryId: String,      // FK a Category
    val icon: String?,           // Icono asociado
    val isFavorite: Boolean,     // Marcado como favorito
    val createdAt: Long,         // Timestamp creación
    val updatedAt: Long          // Timestamp actualización
)
```

#### Category
```kotlin
@Serializable
data class Category(
    val id: String,
    val name: String,
    val color: Int,              // Color ARGB
    val icon: String,            // Nombre del icono
    val isCustom: Boolean,       // true = creada por usuario
    val isDeletable: Boolean     // false = predefinida
)
```

## Capa de Datos

### Responsabilidades
- Implementación de repositorios
- Persistencia en base de datos (Room)
- Almacenamiento seguro (Android Keystore)
- Cifrado/descifrado de datos sensibles

### Componentes

#### Repositorios (Impl)
Implementan las interfaces definidas en Domain:

```kotlin
class PasswordRepositoryImpl(
    private val passwordEntryDao: PasswordEntryDao,
    private val dataCipher: DataCipher,
    private val authRepository: AuthRepository
) : PasswordRepository {
    
    override suspend fun getAllEntries(): Flow<List<PasswordEntry>> {
        return passwordEntryDao.getAll().map { entities ->
            entities.mapNotNull { decryptEntry(it) }
        }
    }
}
```

#### Room Database

**Entidades:**
- `PasswordEntryEntity` - Entradas de contraseñas
- `CategoryEntity` - Categorías
- `SettingsEntity` - Configuración clave-valor

**DAOs:**
- `PasswordEntryDao` - Operaciones CRUD de contraseñas
- `CategoryDao` - Operaciones CRUD de categorías
- `SettingsDao` - Operaciones CRUD de configuración

#### Componentes de Seguridad

| Componente | Responsabilidad |
|------------|-----------------|
| `KeystoreManager` | Gestión de claves RSA en Android Keystore |
| `PasswordDeriver` | Derivación de clave con PBKDF2-HMAC-SHA256 |
| `CipherManager` | Cifrado RSA con clave del Keystore |
| `DataCipher` | Cifrado AES-GCM de campos sensibles |
| `SecureStorage` | Almacenamiento de metadata en SharedPreferences |
| `BiometricAuthenticator` | Autenticación biométrica |

### Flujo de Cifrado

```
1. Usuario ingresa password maestro
           ↓
2. PasswordDeriver.deriveKey() → Clave de 256 bits (PBKDF2)
           ↓
3. CipherManager.encrypt() → Cifra clave con RSA del Keystore
           ↓
4. SecureStorage.saveEncryptedKey() → Guarda clave cifrada
           ↓
5. Para cifrar datos:
   - DataCipher.encrypt() → AES-GCM con clave derivada
   - Base64.encode → Guarda en Room
```

## Inyección de Dependencias (Koin)

### Módulos

El módulo principal `AppModule` define:

1. **Security Layer** (singletons)
   ```kotlin
   single { KeystoreManager() }
   single { PasswordDeriver() }
   single { CipherManager() }
   single { DataCipher() }
   single { SecureStorage(androidContext()) }
   single { BiometricAuthenticator() }
   ```

2. **Database** (singletons)
   ```kotlin
   single { PasswordDatabase.getInstance(androidContext()) }
   single { get<PasswordDatabase>().passwordEntryDao() }
   single { get<PasswordDatabase>().categoryDao() }
   single { get<PasswordDatabase>().settingsDao() }
   ```

3. **Repositories** (interfaces → implementaciones)
   ```kotlin
   single<AuthRepository> { AuthRepositoryImpl(...) }
   single<PasswordRepository> { PasswordRepositoryImpl(...) }
   single<CategoryRepository> { CategoryRepositoryImpl(get()) }
   single<SettingsRepository> { SettingsRepositoryImpl(get()) }
   ```

4. **Use Cases** (singletons)
   ```kotlin
   single { GetAllPasswords(get()) }
   single { CreatePasswordEntry(get()) }
   // ... todos los casos de uso
   ```

5. **ViewModels** (scoped por viewModel)
   ```kotlin
   viewModel { PasswordListViewModel(get(), get(), get()) }
   viewModel { PasswordFormViewModel(get(), get(), get(), get(), get()) }
   ```

## Navegación

El grafo de navegación utiliza **Navigation Compose**:

```kotlin
sealed class Screen(val route: String) {
    object Login : Screen("login")
    object Home : Screen("home")
    object PasswordList : Screen("password_list")
    object PasswordDetail : Screen("password_detail/{id}")
    // ... más rutas
}

@Composable
fun NavGraph() {
    val navController = rememberNavController()
    
    NavHost(navController = navController, startDestination = "login") {
        composable("login") { LoginScreen(...) }
        composable("home") { HomeScreen(...) }
        composable("password_detail/{id}") { backStackEntry ->
            val id = backStackEntry.arguments?.getString("id")
            PasswordDetailScreen(...)
        }
    }
}
```

## Patrones de Diseño Utilizados

| Patrón | Uso |
|--------|-----|
| **Repository** | Abstracción de fuentes de datos |
| **Use Case** | Encapsula lógica de negocio específica |
| **MVVM** | Separación UI-lógica |
| **Singleton** | Instancias únicas (Database, Security) |
| **Observer** | Flow/StateFlow para reactividad |
| **Dependency Injection** | Inversión de dependencias con Koin |
| **Strategy** | Diferentes algoritmos de cifrado |

## Principios de Diseño

### SOLID

1. **Single Responsibility**: Cada clase tiene una única responsabilidad
   - `PasswordEntryDao` solo maneja acceso a BD
   - `DataCipher` solo maneja cifrado AES-GCM
   - `PasswordListViewModel` solo maneja estado de lista

2. **Open/Closed**: Abierto para extensión, cerrado para modificación
   - Nuevos casos de uso se agregan sin modificar existentes
   - Nuevas pantallas no modifican navegación existente

3. **Liskov Substitution**: Interfaces implementadas correctamente
   - `PasswordRepositoryImpl` implementa `PasswordRepository`
   - ViewModels pueden reemplazarse sin afectar Views

4. **Interface Segregation**: Interfaces pequeñas y específicas
   - `AuthRepository` solo métodos de autenticación
   - `CategoryRepository` solo métodos de categorías

5. **Dependency Inversion**: Dependencia de abstracciones
   - ViewModels dependen de casos de uso (interfaces)
   - Casos de uso dependen de interfaces de repositorio

## Flujo de Inicio de la Aplicación

```
Application.onCreate()
       ↓
1. Inicializar Koin (DI)
       ↓
2. SeedPredefinedCategories (si es primera vez)
       ↓
3. Verificar si hay password maestra configurada
       ↓
4. Navegar a Login u Onboarding
       ↓
5. Usuario autentica → Se obtiene clave maestra
       ↓
6. Navegar a Home/PasswordList
```

## Manejo de Estados

Los ViewModels utilizan **StateFlow** para manejar estados:

```kotlin
// Estado inicial
private val _state = MutableStateFlow(PasswordListState(isLoading = true))

// Observar cambios en la UI
val state by viewModel.state.collectAsState()

// Actualizar estado
_state.value = _state.value.copy(entries = newList, isLoading = false)
```

## Testing

### Unit Tests (Dominio y Data)
```kotlin
@Test
fun `password deriver should generate same key from same password`() {
    // Given
    val password = "testPassword123".toCharArray()
    val salt = byteArrayOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16)
    
    // When
    val key1 = PasswordDeriver().deriveKey(password, salt)
    val key2 = PasswordDeriver().deriveKey(password, salt)
    
    // Then
    assertContentEquals(key1, key2)
}
```

### UI Tests (Compose)
```kotlin
@Test
fun passwordListScreen_displaysEntries() {
    composeTestRule.setContent {
        PasswordListScreen(
            viewModel = fakeViewModel,
            onNavigateToDetail = {},
            onNavigateToCreate = {},
            onNavigateToEdit = {},
            categories = emptyList()
        )
    }
    
    composeTestRule
        .onNodeWithText("Mi Contraseña")
        .assertIsDisplayed()
}
```

---

**Documentación Relacionada:**
- [Capa de Datos](../data/overview.md)
- [Capa de Dominio](../domain/overview.md)
- [Capa de Presentación](../presentation/overview.md)
- [Seguridad](../security/overview.md)
