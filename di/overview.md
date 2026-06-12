# Inyección de Dependencias con Koin

## Visión General

Password Manager utiliza **Koin** como framework de inyección de dependencias. Koin es una solución ligera escrita en Kotlin que proporciona:

- Inyección sin proxies ni generación de código
- Integración sencilla con Jetpack Compose
- Scope management (singleton, viewModel, etc.)
- Fácil testing con mocks

---

## Archivos de Documentación

| Archivo | Contenido |
|---------|-----------|
| [`modules.md`](modules.md) | Configuración completa del módulo appModule: Security, Database, DataStore, Repositories, Use Cases, ViewModels |
| [`usage.md`](usage.md) | Uso práctico: inyección en Compose, Activities, Fragments, ViewModels, testing y errores comunes |

---

## Resumen de Tipos de Binding

| Tipo | Scope | Uso |
|------|-------|-----|
| `single` | Aplicación | Repositorios, Use Cases, servicios |
| `factory` | Nueva instancia | Objetos ligeros sin estado |
| `viewModel` | ViewModel | Exclusivo para ViewModels |

---

## Componentes Principales

### Security Layer (6)
`KeystoreManager`, `PasswordDeriver`, `CipherManager`, `DataCipher`, `SecureStorage`, `BiometricAuthenticator`

### Database
`PasswordDatabase` + DAOs: `PasswordEntryDao`, `CategoryDao`, `SettingsDao`

### Repositories (4)
`AuthRepository`, `PasswordRepository`, `CategoryRepository`, `SettingsRepository`

### Use Cases (25+)
- Auth: `SetupMasterPassword`, `AuthenticateUser`, `ChangeMasterPassword`
- Password: `GetAllPasswords`, `CreatePasswordEntry`, `DeletePasswordEntry`, etc.
- Category: `GetCategories`, `CreateCategory`, `DeleteCategory`
- Settings: `GetSettings`, `SetThemeMode`, `SetBiometricEnabled`
- Backup: `ExportEncryptedBackup`, `ImportFromCSV`
- Audit: `AuditWeakPasswords`
- Statistics: `GetSecurityStatistics`

### ViewModels (10)
`AuthViewModel`, `PasswordListViewModel`, `PasswordFormViewModel`, `PasswordDetailViewModel`, `PasswordGeneratorViewModel`, `CategoryManagementViewModel`, `SettingsViewModel`, `BackupViewModel`, `AuditViewModel`, `StatisticsViewModel`

---

## Uso en Compose

```kotlin
@Composable
fun PasswordListScreen(
    viewModel: PasswordListViewModel = koinViewModel()
) {
    val state by viewModel.state.collectAsState()
    // UI
}
```

## Uso en Activities

```kotlin
class MainActivity : ComponentActivity() {
    private val autoLockManager: AutoLockManager by inject()
}
```

---

## Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| `InstanceCreationException` | Error en inicialización | Verificar dependencias |
| `NoDefinitionFoundException` | Tipo no registrado | Añadir definición al módulo |
| `DependencyCycleException` | Ciclo circular | Reestructurar dependencias |
| `ScopeNotStartedException` | Scope no iniciado | Asegurar scope activo |
