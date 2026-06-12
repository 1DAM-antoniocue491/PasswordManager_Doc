# Inyección de Dependencias con Koin

## Visión General

Password Manager utiliza **Koin** como framework de inyección de dependencias. Koin es una solución ligera escrita en Kotlin que proporciona:

- Inyección sin proxies ni generación de código
- Integración sencilla con Jetpack Compose
- Scope management (singleton, viewModel, etc.)
- Fácil testing con mocks

## Configuración

### Dependencias (build.gradle.kts)

```kotlin
dependencies {
    // Koin Core
    implementation("io.insert-koin:koin-core:3.5.3")
    implementation("io.insert-koin:koin-android:3.5.3")
    implementation("io.insert-koin:koin-androidx-compose:3.5.3")
    
    // Koin Annotations (opcional)
    implementation("io.insert-koin:koin-annotations:1.3.1")
    ksp("io.insert-koin:koin-ksp-compiler:1.3.1")
}
```

### Application

```kotlin
class PasswordManagerApplication : Application() {

    private lateinit var autoLockManager: AutoLockManager

    override fun onCreate() {
        super.onCreate()

        // Inicializar Koin
        startKoin {
            androidLogger()
            androidContext(this@PasswordManagerApplication)
            modules(appModule)
        }

        // Obtener dependencias inicializadas
        autoLockManager = get<AutoLockManager>()
        autoLockManager.initialize()
    }
}
```

## Módulo Principal (AppModule)

### Estructura

El módulo principal está organizado por capas:

```kotlin
val appModule = module {
    // 1. Security Layer
    // 2. Database
    // 3. DataStore
    // 4. Repositories
    // 5. Use Cases
    // 6. ViewModels
    // 7. Other
}
```

### 1. Security Layer

```kotlin
val appModule = module {
    // Componentes de seguridad (singletons)
    single { KeystoreManager() }
    single { PasswordDeriver() }
    single { CipherManager() }
    single { DataCipher() }
    single { SecureStorage(androidContext()) }
    single { BiometricAuthenticator() }
}
```

**Características**:
- Todos son `single` (instancia única)
- Sin dependencias entre ellos (excepto SecureStorage que necesita Context)
- Se inicializan en orden de dependencia

### 2. Database

```kotlin
single { PasswordDatabase.getInstance(androidContext()) }
single { get<PasswordDatabase>().passwordEntryDao() }
single { get<PasswordDatabase>().categoryDao() }
single { get<PasswordDatabase>().settingsDao() }
```

**Patrón**:
- Database como singleton
- DAOs se obtienen de la instancia de Database

### 3. DataStore

```kotlin
// DataStore para settings
private val Context.dataStore by preferencesDataStore(name = "settings")

single { androidContext().dataStore }
```

### 4. Repositories

```kotlin
// AuthRepository
single<AuthRepository> {
    AuthRepositoryImpl(
        keystoreManager = get(),
        passwordDeriver = get(),
        cipherManager = get(),
        dataCipher = get(),
        secureStorage = get()
    )
}

// PasswordRepository
single<PasswordRepository> {
    PasswordRepositoryImpl(
        passwordEntryDao = get(),
        dataCipher = get(),
        authRepository = get()
    )
}

// CategoryRepository
single<CategoryRepository> {
    CategoryRepositoryImpl(get()) // CategoryDao
}

// SettingsRepository
single<SettingsRepository> {
    SettingsRepositoryImpl(get()) // DataStore
}
```

**Patrón**:
- Interface binding: `single<Interface> { Implementación() }`
- Dependencias resueltas automáticamente con `get()`

### 5. Use Cases

#### Auth Use Cases

```kotlin
single { SetupMasterPassword(get()) }  // AuthRepository
single { AuthenticateUser(get()) }      // AuthRepository
single { ChangeMasterPassword(get(), get(), get()) }  // AuthRepository, PasswordRepository, CipherManager
```

#### Password Use Cases

```kotlin
single { GetAllPasswords(get()) }           // PasswordRepository
single { GetPasswordById(get()) }           // PasswordRepository
single { CreatePasswordEntry(get()) }       // PasswordRepository
single { UpdatePasswordEntry(get()) }       // PasswordRepository
single { DeletePasswordEntry(get()) }       // PasswordRepository
single { SearchPasswords(get()) }           // PasswordRepository
single { FilterPasswordsByCategory(get()) } // PasswordRepository
single { ToggleFavoriteEntry(get()) }       // PasswordRepository
single { GeneratePassword() }               // Sin dependencias
```

#### Category Use Cases

```kotlin
single { SeedPredefinedCategories(get()) }  // CategoryDao
single { GetCategories(get()) }             // CategoryRepository
single { CreateCategory(get()) }            // CategoryRepository
single { UpdateCategory(get()) }            // CategoryRepository
single { DeleteCategory(get()) }            // CategoryRepository
```

#### Settings Use Cases

```kotlin
single { GetSettings(get()) }               // SettingsRepository
single { UpdateSettings(get()) }            // SettingsRepository
single { GetThemeMode(get()) }              // SettingsRepository
single { SetThemeMode(get()) }              // SettingsRepository
single { IsBiometricEnabled(get()) }        // SettingsRepository
single { SetBiometricEnabled(get()) }       // SettingsRepository
single { GetLockTimeout(get()) }            // SettingsRepository
single { SetLockTimeout(get()) }            // SettingsRepository
```

#### Backup Use Cases

```kotlin
single { ExportEncryptedBackup(get(), get(), get(), get(), androidContext()) }
single { ImportFromCSV(get(), androidContext()) }
```

#### Audit & Statistics

```kotlin
single { AuditWeakPasswords(get()) }        // PasswordRepository
single { GetSecurityStatistics(get(), get(), get()) }
```

### 6. ViewModels

```kotlin
// Auth
viewModel {
    AuthViewModel(
        setupMasterPassword = get(),
        authenticateUser = get(),
        seedPredefinedCategories = get(),
        authRepository = get(),
        autoLockManager = get(),
        application = get()
    )
}

viewModel { ChangePasswordViewModel(get()) }

// Password List
viewModel {
    PasswordListViewModel(
        getAllPasswords = get(),
        deletePasswordEntry = get(),
        toggleFavoriteEntry = get()
    )
}

// Password Detail
viewModel {
    PasswordDetailViewModel(
        getPasswordById = get(),
        context = get()
    )
}

// Password Form
viewModel { PasswordFormViewModel(get(), get(), get(), get(), get()) }

// Password Generator
viewModel { PasswordGeneratorViewModel(get()) }

// Category Management
viewModel { CategoryManagementViewModel(get(), get(), get(), get()) }

// Backup
viewModel {
    BackupViewModel(
        exportBackupUseCase = get(),
        importCsvUseCase = get(),
        application = get()
    )
}

// Settings
viewModel { SettingsViewModel(get(), get(), get(), get(), get()) }

// Audit
viewModel { AuditViewModel(get()) }

// Statistics
viewModel { StatisticsViewModel(get()) }
```

**Scope**:
- `viewModel` crea una instancia por ViewModel lifecycle
- Se limpia automáticamente cuando el ViewModel se destruye

### 7. Other

```kotlin
single { AutoLockManager(get(), get()) }  // SettingsRepository, Application
```

---

## Tipos de Binding

### single

Instancia única compartida en toda la aplicación:

```kotlin
single { KeystoreManager() }
single<AuthRepository> { AuthRepositoryImpl(...) }
```

**Cuándo usar**:
- Servicios sin estado
- Repositorios
- Use Cases
- Componentes de seguridad

### factory

Nueva instancia cada vez que se solicita:

```kotlin
factory { PasswordDeriver() }
```

**Cuándo usar**:
- Objetos ligeros sin estado compartido
- Cuando se necesita una instancia nueva

### viewModel

Instancia vinculada al lifecycle del ViewModel:

```kotlin
viewModel { PasswordListViewModel(get(), get()) }
```

**Cuándo usar**:
- Exclusivo para ViewModels
- Se limpia automáticamente

---

## Inyección en Compose

### koinViewModel

```kotlin
@Composable
fun PasswordListScreen(
    viewModel: PasswordListViewModel = koinViewModel(),
    onNavigateToDetail: (String) -> Unit,
    // ...
) {
    val state by viewModel.state.collectAsState()
    // ...
}
```

### koinInject

Para otras dependencias en Composables:

```kotlin
@Composable
fun SomeComponent(
    repository: PasswordRepository = koinInject()
) {
    // Usar repository
}
```

### getKoin

Para acceso manual fuera de Compose:

```kotlin
val koin = getKoin()
val repository = koin.get<PasswordRepository>()
```

---

## Inyección en Activities

```kotlin
class MainActivity : ComponentActivity() {

    private val autoLockManager: AutoLockManager by inject()

    override fun onCreate() {
        super.onCreate()
        
        // Usar autoLockManager
        enableEdgeToEdge()
        
        setContent {
            PasswordManagerTheme {
                NavGraph()
            }
        }
    }
}
```

---

## Inyección en Fragments

```kotlin
class BiometricFragment : Fragment() {

    private val biometricAuthenticator: BiometricAuthenticator by inject()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        biometricAuthenticator.authenticate(
            fragment = this,
            onAuthSuccess = { /* ... */ },
            onAuthError = { /* ... */ }
        )
    }
}
```

---

## Testing con Koin

### Unit Tests

```kotlin
class PasswordListViewModelTest {

    private lateinit var viewModel: PasswordListViewModel
    private lateinit var getAllPasswords: GetAllPasswords

    @Before
    fun setup() {
        // Start Koin con módulos de test
        startKoin {
            modules(
                module {
                    single { mockk<PasswordRepository>() }
                    factory { GetAllPasswords(get()) }
                    viewModel { 
                        PasswordListViewModel(
                            getAllPasswords = get(),
                            deletePasswordEntry = get(),
                            toggleFavoriteEntry = get()
                        )
                    }
                }
            )
        }
        
        getAllPasswords = getKoin().get()
    }

    @After
    fun teardown() {
        stopKoin()
    }

    @Test
    fun `test example`() = runTest {
        // Obtener ViewModel de Koin
        viewModel = getKoin().get()
        
        // Test logic...
    }
}
```

### Tests sin Koin

Para tests más simples, inyectar manualmente:

```kotlin
class PasswordListViewModelTest {

    private val getAllPasswords = mockk<GetAllPasswords>()
    private val deletePasswordEntry = mockk<DeletePasswordEntry>()
    private val toggleFavoriteEntry = mockk<ToggleFavoriteEntry>()

    private lateinit var viewModel: PasswordListViewModel

    @Before
    fun setup() {
        // Inyección manual sin Koin
        viewModel = PasswordListViewModel(
            getAllPasswords,
            deletePasswordEntry,
            toggleFavoriteEntry
        )
    }

    @Test
    fun `should filter entries when search query changes`() = runTest {
        // Test logic...
    }
}
```

---

## Módulos Separados

Para proyectos grandes, separar en módulos:

```kotlin
val securityModule = module {
    single { KeystoreManager() }
    single { PasswordDeriver() }
    single { CipherManager() }
    single { DataCipher() }
}

val databaseModule = module {
    single { PasswordDatabase.getInstance(androidContext()) }
    single { get<PasswordDatabase>().passwordEntryDao() }
    single { get<PasswordDatabase>().categoryDao() }
}

val repositoryModule = module {
    single<AuthRepository> { AuthRepositoryImpl(...) }
    single<PasswordRepository> { PasswordRepositoryImpl(...) }
}

val useCaseModule = module {
    single { GetAllPasswords(get()) }
    single { CreatePasswordEntry(get()) }
    // ...
}

val viewModelModule = module {
    viewModel { PasswordListViewModel(get(), get(), get()) }
    // ...
}

// Combinar en Application
startKoin {
    modules(
        securityModule,
        databaseModule,
        repositoryModule,
        useCaseModule,
        viewModelModule
    )
}
```

---

## Grafo de Dependencias

```
PasswordListViewModel
├── GetAllPasswords
│   └── PasswordRepository
│       └── PasswordRepositoryImpl
│           ├── PasswordEntryDao
│           │   └── PasswordDatabase
│           ├── DataCipher
│           └── AuthRepository
│               └── AuthRepositoryImpl
│                   ├── KeystoreManager
│                   ├── PasswordDeriver
│                   ├── CipherManager
│                   ├── DataCipher
│                   └── SecureStorage
├── DeletePasswordEntry
│   └── PasswordRepository (mismo que arriba)
└── ToggleFavoriteEntry
    └── PasswordRepository (mismo que arriba)
```

**Resolución**:
- Koin resuelve dependencias recursivamente
- Singletons se comparten en todo el grafo
- ViewModels tienen scope propio

---

## Errores Comunes

### 1. NoStartKoin

```
org.koin.core.error.InstanceCreationException: Could not create instance
```

**Solución**: Asegurar que `startKoin` se llama en `Application.onCreate()`

### 2. Dependency Not Found

```
org.koin.core.error.NoDefinitionFoundException: No definition found for type X
```

**Solución**: Verificar que la dependencia está declarada en el módulo

### 3. Circular Dependency

```
org.koin.core.error.DependencyCycleException
```

**Solución**: Refactorizar para eliminar ciclo

### 4. Wrong Scope

```
org.koin.core.error.ScopeNotStartedException
```

**Solución**: Asegurar que el scope está activo cuando se usa

---

## Mejores Prácticas

### 1. Interface Binding

```kotlin
// ✅ CORRECTO
single<PasswordRepository> { PasswordRepositoryImpl(get()) }

// ❌ INCORRECTO
single { PasswordRepositoryImpl(get()) }  // No se puede inyectar la interface
```

### 2. Orden de Declaración

```kotlin
// ✅ CORRECTO - Dependencias antes que dependientes
single { KeystoreManager() }
single { PasswordDeriver() }
single { CipherManager() }
single<AuthRepository> { AuthRepositoryImpl(get(), get(), get()) }
```

### 3. Usar Parámetros Nombrados

```kotlin
// ✅ LEGIBLE
single<AuthRepository> {
    AuthRepositoryImpl(
        keystoreManager = get(),
        passwordDeriver = get(),
        cipherManager = get()
    )
}

// ❌ CONFUSO
single<AuthRepository> { 
    AuthRepositoryImpl(get(), get(), get()) 
}
```

### 4. Modularizar

```kotlin
// Para proyectos grandes
val appModule = module {
    includes(securityModule, databaseModule, repositoryModule, useCaseModule, viewModelModule)
}
```

### 5. Lazy Injection

```kotlin
// Para dependencias pesadas o opcionales
class MyViewModel(
    private val heavyDependency: Lazy<HeavyService>
) : ViewModel() {
    
    fun doSomething() {
        // Se inicializa solo cuando se usa
        heavyDependency.value.doWork()
    }
}
```

## Configuración de Koin

### PasswordManagerApplication

**Archivo**: `PasswordManagerApplication.kt`

```kotlin
class PasswordManagerApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        instance = this
        
        // Iniciar Koin
        startKoin()
    }
    
    private fun startKoin() {
        KoinApplication.init()
        
        startKoin {
            androidContext(this@PasswordManagerApplication)
            modules(appModule)
        }
        
        // Obtener AutoLockManager después de que Koin esté listo
        val autoLockManager = koin.get<AutoLockManager>()
        autoLockManager.initialize()
    }
    
    companion object {
        lateinit var instance: PasswordManagerApplication private set
    }
}
```

**Flujo de Inicialización**:
1. `Application.onCreate()` es llamado al iniciar la app
2. Se inicializa Koin con `appModule`
3. Se obtiene `AutoLockManager` del contenedor Koin
4. Se llama a `autoLockManager.initialize()` para registrar lifecycle observer

---

## Inyección en Compose

### koinViewModel()

Para obtener ViewModels en composables:

```kotlin
@Composable
fun PasswordListScreen(
    onNavigateToDetail: (String) -> Unit,
    onNavigateToCreate: () -> Unit,
    onNavigateToEdit: (String) -> Unit
) {
    // Inyección automática desde Koin
    val viewModel: PasswordListViewModel = koinViewModel()
    
    val state by viewModel.state.collectAsStateWithLifecycle()
    
    // UI...
}
```

### koinInjectable()

Para otras dependencias en composables:

```kotlin
@Composable
fun SomeScreen() {
    val dataCipher: DataCipher = koinInjectable()
    val settingsRepository: SettingsRepository = koinInjectable()
    
    // Usar dependencias...
}
```

---

## Árbol de Dependencias

```
appModule
├── Security
│   ├── KeystoreManager
│   ├── PasswordDeriver
│   ├── CipherManager
│   ├── DataCipher
│   ├── SecureStorage (context)
│   └── BiometricAuthenticator
│
├── Database
│   ├── PasswordDatabase (context)
│   ├── PasswordEntryDao (de Database)
│   ├── CategoryDao (de Database)
│   └── SettingsDao (de Database)
│
├── DataStore
│   └── dataStore (context)
│
├── Repositories
│   ├── AuthRepository → AuthRepositoryImpl
│   ├── PasswordRepository → PasswordRepositoryImpl
│   ├── CategoryRepository → CategoryRepositoryImpl
│   └── SettingsRepository → SettingsRepositoryImpl
│
├── Use Cases - Auth
│   ├── SetupMasterPassword (AuthRepository)
│   ├── AuthenticateUser (AuthRepository)
│   └── ChangeMasterPassword (AuthRepository, PasswordRepository, CipherManager)
│
├── Use Cases - Password
│   ├── GetAllPasswords (PasswordRepository)
│   ├── GetPasswordById (PasswordRepository)
│   ├── CreatePasswordEntry (PasswordRepository)
│   ├── UpdatePasswordEntry (PasswordRepository)
│   ├── DeletePasswordEntry (PasswordRepository)
│   ├── SearchPasswords (PasswordRepository)
│   ├── FilterPasswordsByCategory (PasswordRepository)
│   ├── ToggleFavoriteEntry (PasswordRepository)
│   └── GeneratePassword ()
│
├── Use Cases - Category
│   ├── SeedPredefinedCategories (CategoryDao)
│   ├── GetCategories (CategoryRepository)
│   ├── CreateCategory (CategoryRepository)
│   ├── UpdateCategory (CategoryRepository)
│   └── DeleteCategory (CategoryRepository)
│
├── Use Cases - Settings
│   ├── GetSettings (SettingsRepository)
│   ├── UpdateSettings (SettingsRepository)
│   ├── GetThemeMode (SettingsRepository)
│   ├── SetThemeMode (SettingsRepository)
│   ├── IsBiometricEnabled (SettingsRepository)
│   ├── SetBiometricEnabled (SettingsRepository)
│   ├── GetLockTimeout (SettingsRepository)
│   └── SetLockTimeout (SettingsRepository)
│
├── Use Cases - Backup
│   ├── ExportEncryptedBackup (PasswordRepository, CategoryRepository, DataCipher, AuthRepository, context)
│   └── ImportFromCSV (PasswordRepository, context)
│
├── Use Cases - Audit/Statistics
│   ├── AuditWeakPasswords (PasswordRepository)
│   └── GetSecurityStatistics (PasswordRepository, CategoryRepository, AuditWeakPasswords)
│
├── ViewModels
│   ├── AuthViewModel (SetupMasterPassword, AuthenticateUser, SeedPredefinedCategories, AuthRepository, AutoLockManager, application)
│   ├── ChangePasswordViewModel (ChangeMasterPassword)
│   ├── PasswordListViewModel (GetAllPasswords, DeletePasswordEntry, ToggleFavoriteEntry)
│   ├── PasswordDetailViewModel (GetPasswordById, context)
│   ├── PasswordFormViewModel (GetAllPasswords, CreatePasswordEntry, UpdatePasswordEntry, GetCategories, GeneratePassword)
│   ├── PasswordGeneratorViewModel (GeneratePassword)
│   ├── CategoryManagementViewModel (GetCategories, CreateCategory, UpdateCategory, DeleteCategory)
│   ├── BackupViewModel (ExportEncryptedBackup, ImportFromCSV, application)
│   ├── SettingsViewModel (GetSettings, UpdateSettings, SetLockTimeout, SetBiometricEnabled, SetThemeMode)
│   ├── AuditViewModel (AuditWeakPasswords)
│   └── StatisticsViewModel (GetSecurityStatistics)
│
└── Other
    └── AutoLockManager (SettingsRepository)
```

---

## Testing con Koin

### Unit Tests con Koin

```kotlin
class PasswordListViewModelTest {
    
    private lateinit var koinApp: KoinApplication
    private lateinit var viewModel: PasswordListViewModel
    
    @Before
    fun setup() {
        koinApp = startKoin {
            modules(
                module {
                    // Mocks para testing
                    single { mockk<GetAllPasswords>() }
                    single { mockk<DeletePasswordEntry>() }
                    single { mockk<ToggleFavoriteEntry>() }
                    
                    // ViewModel bajo test
                    viewModel { PasswordListViewModel(get(), get(), get()) }
                }
            )
        }
        
        viewModel = koinApp.koin.get()
    }
    
    @After
    fun teardown() {
        koinApp.close()
    }
    
    @Test
    fun `should load passwords on init`() = runTest {
        // Given
        val mockPasswords = listOf(
            PasswordEntry(id = "1", title = "Test", ...)
        )
        val flow = flowOf(mockPasswords)
        every { get<GetAllPasswords>().invoke() } returns flow
        
        // When
        viewModel // Inicializa
        
        // Then
        val state = viewModel.state.value
        assertEquals(mockPasswords, state.entries)
    }
}
```

### Koin para Tests de Instrumentación

```kotlin
@RunWith(AndroidJUnit4::class)
class InstrumentationTest {
    
    @get:Rule
    val koinRule = KoinAndroidRule {
        modules(appModule)
    }
    
    @Test
    fun testDatabaseInjection() {
        val database: PasswordDatabase = koinRule.koin.get()
        assertNotNull(database)
    }
}
```

---

## Consideraciones de Rendimiento

### Single vs Factory

```kotlin
// ✅ SINGLE: Una instancia para toda la vida de la app
single { PasswordDeriver() }  // Sin estado, seguro
single { DataCipher() }       // Sin estado, seguro
single { get<PasswordDatabase>().passwordEntryDao() }  // DAO es thread-safe

// ✅ FACTORY (default): Nueva instancia cada vez
// factory { SomeClassWithState() }

// ⚠️ VIEWMODEL: Scoped al lifecycle
viewModel { PasswordListViewModel(get(), get(), get()) }
```

### Lazy Injection

```kotlin
// Lazy: Se crea solo cuando se usa
single { lazy { ExpensiveResource() } }

// En el ViewModel
class MyViewModel(private val resource: Lazy<ExpensiveResource>) : ViewModel() {
    fun doSomething() {
        resource.value.doWork()  // Se crea aquí
    }
}
```

---

**Documentación Relacionada:**
- [Arquitectura](../arquitectura/overview.md)
- [Testing Guide](../testing-guide.md)
- [Koin Documentation](https://insert-koin.io/docs/)
