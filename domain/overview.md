# Capa de Dominio

## Visión General

La capa de dominio es el núcleo de la aplicación. Contiene:
- **Modelos de negocio**: Representan las entidades del dominio
- **Casos de uso**: Encapsulan lógica de negocio específica
- **Repositorios (interfaces)**: Contratos para acceso a datos

**Características clave**:
- ✅ Independiente de Android
- ✅ Independiente de frameworks
- ✅ Pure Kotlin
- ✅ Fácilmente testeable

---

## Estructura de Directorios

```
domain/
├── model/                      # Modelos de negocio
│   ├── Category.kt
│   ├── PasswordEntry.kt
│   ├── AppSettings.kt
│   ├── BackupData.kt
│   └── EncryptedPasswordEntry.kt
│
├── repository/                 # Interfaces de repositorio
│   ├── AuthRepository.kt
│   ├── CategoryRepository.kt
│   ├── PasswordRepository.kt
│   └── SettingsRepository.kt
│
└── usecase/                    # Casos de uso
    ├── auth/
    │   ├── AuthenticateUser.kt
    │   ├── ChangeMasterPassword.kt
    │   └── SetupMasterPassword.kt
    ├── password/
    │   ├── CreatePasswordEntry.kt
    │   ├── DeletePasswordEntry.kt
    │   ├── GetAllPasswords.kt
    │   ├── GetPasswordById.kt
    │   ├── SearchPasswords.kt
    │   ├── ToggleFavoriteEntry.kt
    │   └── GeneratePassword.kt
    ├── category/
    │   ├── CreateCategory.kt
    │   ├── DeleteCategory.kt
    │   ├── GetCategories.kt
    │   ├── UpdateCategory.kt
    │   └── SeedPredefinedCategories.kt
    ├── settings/
    │   ├── GetSettings.kt
    │   ├── UpdateSettings.kt
    │   └── SettingsUseCases.kt
    ├── backup/
    │   ├── ExportEncryptedBackup.kt
    │   └── ImportFromCSV.kt
    ├── audit/
    │   └── AuditWeakPasswords.kt
    └── statistics/
        └── GetSecurityStatistics.kt
```

---

## Archivos de Documentación

| Archivo | Contenido |
|---------|-----------|
| [`models.md`](models.md) | Modelos: PasswordEntry, Category, AppSettings, BackupData, EncryptedPasswordEntry, WeakPasswordEntry |
| [`repositories.md`](repositories.md) | Interfaces: PasswordRepository, AuthRepository, CategoryRepository, SettingsRepository |
| [`usecases.md`](usecases.md) | Todos los casos de uso organizados por funcionalidad |

---

## Resumen de Modelos

### PasswordEntry
Representa una entrada de contraseña con todos sus campos (id, title, username, password, url, notes, categoryId, icon, isFavorite, createdAt, updatedAt).

### Category
Categoría de organización con color e icono. Puede ser predefinida (no eliminable) o personalizada.

### AppSettings
Configuración de la aplicación: tema, biometría, timeout de bloqueo.

### BackupData
Estructura para exportar/importar copias de seguridad cifradas.

---

## Resumen de Repositorios

| Repositorio | Responsabilidad |
|-------------|-----------------|
| `PasswordRepository` | CRUD de contraseñas, búsqueda, filtrado |
| `AuthRepository` | Gestión de contraseña maestra y autenticación |
| `CategoryRepository` | CRUD de categorías |
| `SettingsRepository` | Configuración de la aplicación |

---

## Resumen de Casos de Uso

### Autenticación (3)
- `SetupMasterPassword`: Configura contraseña inicial
- `AuthenticateUser`: Autentica con password/biometría
- `ChangeMasterPassword`: Cambia contraseña y re-cifra datos

### Gestión de Contraseñas (8)
- `GetAllPasswords`, `GetPasswordById`, `SearchPasswords`
- `CreatePasswordEntry`, `UpdatePasswordEntry`, `DeletePasswordEntry`
- `ToggleFavoriteEntry`, `GeneratePassword`

### Gestión de Categorías (5)
- `GetCategories`, `CreateCategory`, `UpdateCategory`, `DeleteCategory`
- `SeedPredefinedCategories`

### Configuración (8)
- `GetSettings`, `UpdateSettings`
- `GetThemeMode`, `SetThemeMode`
- `IsBiometricEnabled`, `SetBiometricEnabled`
- `GetLockTimeout`, `SetLockTimeout`

### Backup (2)
- `ExportEncryptedBackup`: Exporta datos cifrados
- `ImportFromCSV`: Importa desde CSV

### Auditoría y Estadísticas (2)
- `AuditWeakPasswords`: Detecta contraseñas débiles
- `GetSecurityStatistics`: Estadísticas de seguridad

---

## Excepciones de Dominio

| Excepción | Descripción |
|-----------|-------------|
| `EmptyTitleException` | Título vacío en formulario |
| `EmptyUsernameException` | Username vacío |
| `EmptyPasswordException` | Contraseña vacía |
| `EmptyCategoryNameException` | Nombre de categoría vacío |
| `WeakPasswordException` | Contraseña maestra débil |
| `CannotDeleteSystemCategoryException` | Intento de eliminar categoría predefinida |
| `CannotModifySystemCategoryException` | Intento de modificar categoría predefinida |
