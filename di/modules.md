# Módulos Koin

Configuración detallada de los módulos de inyección de dependencias.

---

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

---

## Application

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

---

## Estructura del Módulo Principal

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

---

## 1. Security Layer

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

---

## 2. Database

```kotlin
single { PasswordDatabase.getInstance(androidContext()) }
single { get<PasswordDatabase>().passwordEntryDao() }
single { get<PasswordDatabase>().categoryDao() }
single { get<PasswordDatabase>().settingsDao() }
```

**Patrón**:
- Database como singleton
- DAOs se obtienen de la instancia de Database

---

## 3. DataStore

```kotlin
// DataStore para settings
private val Context.dataStore by preferencesDataStore(name = "settings")

single { androidContext().dataStore }
```

---

## 4. Repositories

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

---

## 5. Use Cases

### Auth Use Cases

```kotlin
single { SetupMasterPassword(get()) }  // AuthRepository
single { AuthenticateUser(get()) }      // AuthRepository
single { ChangeMasterPassword(get(), get(), get()) }  // AuthRepository, PasswordRepository, CipherManager
```

### Password Use Cases

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

### Category Use Cases

```kotlin
single { SeedPredefinedCategories(get()) }  // CategoryDao
single { GetCategories(get()) }             // CategoryRepository
single { CreateCategory(get()) }            // CategoryRepository
single { UpdateCategory(get()) }            // CategoryRepository
single { DeleteCategory(get()) }            // CategoryRepository
```

### Settings Use Cases

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

### Backup Use Cases

```kotlin
single { ExportEncryptedBackup(get(), get(), get(), get(), androidContext()) }
single { ImportFromCSV(get(), androidContext()) }
```

### Audit & Statistics

```kotlin
single { AuditWeakPasswords(get()) }        // PasswordRepository
single { GetSecurityStatistics(get(), get(), get()) }
```

---

## 6. ViewModels

### Auth

```kotlin
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
```

### Password List

```kotlin
viewModel {
    PasswordListViewModel(
        getAllPasswords = get(),
        deletePasswordEntry = get(),
        toggleFavoriteEntry = get()
    )
}
```

### Password Detail

```kotlin
viewModel {
    PasswordDetailViewModel(
        getPasswordById = get(),
        context = get()
    )
}
```

### Password Form

```kotlin
viewModel { PasswordFormViewModel(get(), get(), get(), get(), get()) }
```

### Password Generator

```kotlin
viewModel { PasswordGeneratorViewModel(get()) }
```

### Category Management

```kotlin
viewModel { CategoryManagementViewModel(get(), get(), get(), get()) }
```

### Backup

```kotlin
viewModel {
    BackupViewModel(
        exportBackupUseCase = get(),
        importCsvUseCase = get(),
        application = get()
    )
}
```

### Settings

```kotlin
viewModel { SettingsViewModel(get(), get(), get(), get(), get()) }
```

### Audit

```kotlin
viewModel { AuditViewModel(get()) }
```

### Statistics

```kotlin
viewModel { StatisticsViewModel(get()) }
```

**Scope**:
- `viewModel` crea una instancia por ViewModel lifecycle
- Se limpia automáticamente cuando el ViewModel se destruye

---

## 7. Other

```kotlin
single { AutoLockManager(get(), get()) }  // SettingsRepository, Application
```
