# Capa de Datos

## Visión General

La capa de datos es responsable de:
- Gestionar el almacenamiento persistente (Room Database)
- Implementar los repositorios definidos en la capa de dominio
- Manejar el cifrado/descifrado de datos sensibles
- Proveer datos a la capa de dominio mediante Flow reactivo

## Estructura de Directorios

```
data/
├── local/                      # Persistencia local
│   ├── dao/                    # Data Access Objects
│   │   ├── PasswordEntryDao.kt
│   │   ├── CategoryDao.kt
│   │   └── SettingsDao.kt
│   ├── entity/                 # Entidades Room
│   │   ├── PasswordEntryEntity.kt
│   │   ├── CategoryEntity.kt
│   │   └── SettingsEntity.kt
│   ├── converter/              # TypeConverters
│   │   └── DateConverters.kt
│   └── PasswordDatabase.kt     # Configuración de Room
│
├── repository/                 # Implementaciones de repositorios
│   ├── AuthRepositoryImpl.kt
│   ├── PasswordRepositoryImpl.kt
│   ├── CategoryRepositoryImpl.kt
│   └── SettingsRepositoryImpl.kt
│
└── security/                   # Componentes de seguridad
    ├── BiometricAuthenticator.kt
    ├── CipherManager.kt
    ├── DataCipher.kt
    ├── KeystoreManager.kt
    ├── PasswordDeriver.kt
    └── SecureStorage.kt
```

## Flujo de Datos Completo

```
┌─────────────────────────────────────────────────────────────────┐
│                         UI (Compose)                             │
│  PasswordListScreen observa state.collectAsState()               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      ViewModel                                   │
│  PasswordListViewModel observa getAllPasswords()                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Use Case                                      │
│  GetAllPasswords(repository.getAllEntries())                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Repository Impl                                 │
│  PasswordRepositoryImpl.getAllEntries()                          │
│    → passwordEntryDao.getAll()                                   │
│    → .map { decryptEntry(it) }                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DAO                                        │
│  PasswordEntryDao.getAll()                                       │
│    → SELECT * FROM password_entries                              │
│    → Flow<List<PasswordEntryEntity>>                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Room Database                                 │
│  Emite cuando hay cambios en la tabla                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Archivos de Documentación

| Archivo | Contenido |
|---------|-----------|
| [`database.md`](database.md) | PasswordDatabase, Entidades (PasswordEntryEntity, CategoryEntity, SettingsEntity), TypeConverters, Migraciones |
| [`dao.md`](dao.md) | PasswordEntryDao, CategoryDao, SettingsDao, Testing de DAOs |
| [`repositories.md`](repositories.md) | PasswordRepositoryImpl, AuthRepositoryImpl, CategoryRepositoryImpl, SettingsRepositoryImpl, Transacciones |
| [`security.md`](security.md) | PasswordDeriver, CipherManager, DataCipher, SecureStorage, BiometricAuthenticator, Testing |

---

## Resumen de Componentes

### Base de Datos
- **PasswordDatabase**: Room Database singleton
- **Entidades**: PasswordEntryEntity, CategoryEntity, SettingsEntity
- **DAOs**: Interfaces con consultas @Query y operaciones CRUD

### Repositorios
- **PasswordRepositoryImpl**: Cifrado/descifrado de contraseñas y notas
- **AuthRepositoryImpl**: Gestión de clave maestra y autenticación
- **CategoryRepositoryImpl**: CRUD de categorías
- **SettingsRepositoryImpl**: Preferencias con DataStore

### Seguridad
- **PasswordDeriver**: PBKDF2-HMAC-SHA256, 100,000 iteraciones
- **CipherManager**: RSA-2048 para cifrar clave derivada
- **DataCipher**: AES-256-GCM para campos individuales
- **SecureStorage**: SharedPreferences para salt y clave cifrada
- **BiometricAuthenticator**: BiometricPrompt wrapper
