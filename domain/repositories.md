# Interfaces de Repositorio

Los repositorios definen los contratos para el acceso a datos.

---

## PasswordRepository

**Archivo**: `repository/PasswordRepository.kt`

```kotlin
interface PasswordRepository {
    suspend fun getAllEntries(): Flow<List<PasswordEntry>>
    suspend fun getAllEntriesList(): List<PasswordEntry>
    suspend fun getEntryById(id: String): PasswordEntry?
    suspend fun searchEntries(query: String): Flow<List<PasswordEntry>>
    suspend fun insertEntry(entry: PasswordEntry)
    suspend fun updateEntry(entry: PasswordEntry)
    suspend fun deleteEntry(entry: PasswordEntry)
    suspend fun getEntriesByCategory(categoryId: String): Flow<List<PasswordEntry>>
    suspend fun getFavoriteEntries(): Flow<List<PasswordEntry>>
}
```

### Contratos

| Método | Retorno | Descripción |
|--------|---------|-------------|
| `getAllEntries()` | `Flow<List<PasswordEntry>>` | Emite cada vez que hay cambios en BD |
| `getAllEntriesList()` | `List<PasswordEntry>` | Obtiene lista instantánea (una vez) |
| `getEntryById(id)` | `PasswordEntry?` | Busca por UUID, null si no existe |
| `searchEntries(query)` | `Flow<List<PasswordEntry>>` | Búsqueda en título, username, URL |
| `insertEntry(entry)` | `Unit` | Crea nueva entrada |
| `updateEntry(entry)` | `Unit` | Actualiza entrada existente |
| `deleteEntry(entry)` | `Unit` | Elimina entrada |
| `getEntriesByCategory(categoryId)` | `Flow<List<PasswordEntry>>` | Filtra por categoría |
| `getFavoriteEntries()` | `Flow<List<PasswordEntry>>` | Solo favoritos |

---

## AuthRepository

**Archivo**: `repository/AuthRepository.kt`

```kotlin
interface AuthRepository {
    suspend fun setupMasterPassword(password: CharArray): Result<Unit>
    suspend fun authenticateUser(password: CharArray): Result<ByteArray>
    suspend fun changeMasterPassword(
        oldPassword: CharArray,
        newPassword: CharArray
    ): Result<Unit>
    suspend fun hasMasterPassword(): Boolean
    fun getMasterKey(): ByteArray?
    fun setMasterKey(key: ByteArray)
    fun clearMasterKey()
}
```

### Contratos

| Método | Retorno | Descripción |
|--------|---------|-------------|
| `setupMasterPassword(password)` | `Result<Unit>` | Configura contraseña inicial |
| `authenticateUser(password)` | `Result<ByteArray>` | Autentica, retorna clave si éxito |
| `changeMasterPassword(old, new)` | `Result<Unit>` | Cambia contraseña, re-cifra datos |
| `hasMasterPassword()` | `Boolean` | Verifica si está configurada |
| `getMasterKey()` | `ByteArray?` | Obtiene clave de sesión |
| `setMasterKey(key)` | `Unit` | Establece clave en memoria |
| `clearMasterKey()` | `Unit` | Limpia clave de memoria |

---

## CategoryRepository

**Archivo**: `repository/CategoryRepository.kt`

```kotlin
interface CategoryRepository {
    fun getAllCategories(): Flow<List<Category>>
    suspend fun createCategory(category: Category)
    suspend fun updateCategory(category: Category)
    suspend fun deleteCategory(category: Category)
}
```

### Contratos

| Método | Retorno | Descripción |
|--------|---------|-------------|
| `getAllCategories()` | `Flow<List<Category>>` | Emite lista de categorías (reactivo) |
| `createCategory(category)` | `Unit` | Crea nueva categoría |
| `updateCategory(category)` | `Unit` | Actualiza categoría existente |
| `deleteCategory(category)` | `Unit` | Elimina categoría (solo personalizables) |

---

## SettingsRepository

**Archivo**: `repository/SettingsRepository.kt`

```kotlin
interface SettingsRepository {
    suspend fun getThemeMode(): ThemeMode
    suspend fun setThemeMode(mode: ThemeMode)
    suspend fun isBiometricEnabled(): Boolean
    suspend fun setBiometricEnabled(enabled: Boolean)
    suspend fun getLockTimeout(): Long
    suspend fun setLockTimeout(timeoutMs: Long)
    suspend fun getSettings(): AppSettings
    suspend fun updateSettings(settings: AppSettings)
}
```

### Contratos

| Método | Retorno | Descripción |
|--------|---------|-------------|
| `getThemeMode()` | `ThemeMode` | Obtiene tema actual |
| `setThemeMode(mode)` | `Unit` | Establece tema (AUTO/LIGHT/DARK) |
| `isBiometricEnabled()` | `Boolean` | Verifica si biometría está activa |
| `setBiometricEnabled(enabled)` | `Unit` | Activa/desactiva biometría |
| `getLockTimeout()` | `Long` | Obtiene timeout en milisegundos |
| `setLockTimeout(timeoutMs)` | `Unit` | Establece timeout de bloqueo |
| `getSettings()` | `AppSettings` | Obtiene toda la configuración |
| `updateSettings(settings)` | `Unit` | Actualiza configuración completa |
