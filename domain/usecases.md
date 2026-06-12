# Casos de Uso

Los casos de uso encapsulan la lógica de negocio específica.

## Patrón de Diseño

Todos los casos de uso siguen el patrón:

```kotlin
class UseCaseName(private val dependency: Dependency) {
    suspend operator fun invoke(parameter: Type): Result<ReturnType> {
        // Lógica de negocio
    }
}
```

**Ventajas**:
- `invoke` como operador permite llamar sin nombre de método
- Single Responsibility: una acción por clase
- Fácil de testear con mocks

---

## Casos de Uso - Autenticación

### SetupMasterPassword

Configura la contraseña maestra inicial del usuario.

```kotlin
class SetupMasterPassword(
    private val authRepository: AuthRepository,
    private val seedCategories: SeedPredefinedCategories
) {
    suspend operator fun invoke(password: CharArray): Result<Unit> {
        return try {
            // Validar fortaleza del password
            if (!PasswordValidator.isValid(password)) {
                return Result.failure(WeakPasswordException())
            }
            
            // Configurar password maestro
            authRepository.setupMasterPassword(password)
            
            // Seed de categorías predefinidas
            seedCategories()
            
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

**Validaciones**:
- Mínimo 8 caracteres
- Al menos una mayúscula
- Al menos un número
- Al menos un símbolo especial

---

### AuthenticateUser

Autentica al usuario con su password maestro.

```kotlin
class AuthenticateUser(private val authRepository: AuthRepository) {
    suspend operator fun invoke(password: CharArray): Result<ByteArray> {
        return authRepository.authenticateUser(password)
    }
}
```

**Flujo**:
1. Derivar clave del password ingresado
2. Descifrar clave almacenada (requiere biometría)
3. Comparar claves
4. Retornar clave maestra si coincide

---

### ChangeMasterPassword

Cambia la contraseña maestra del usuario.

```kotlin
class ChangeMasterPassword(
    private val authRepository: AuthRepository,
    private val passwordRepository: PasswordRepository,
    private val cipherManager: CipherManager
) {
    suspend operator fun invoke(
        oldPassword: CharArray,
        newPassword: CharArray
    ): Result<Unit> {
        return try {
            // 1. Verificar password actual
            val authResult = authRepository.authenticateUser(oldPassword)
            if (authResult.isFailure) {
                return Result.failure(authResult.exceptionOrNull()!!)
            }
            
            // 2. Obtener todas las contraseñas
            val entries = passwordRepository.getAllEntriesList()
            
            // 3. Descifrar todas las entradas con clave vieja
            val oldKey = authRepository.getMasterKey()!!
            val decryptedEntries = entries.map { entry ->
                entry.copy(
                    password = decrypt(entry.password, oldKey),
                    notes = entry.notes?.let { decrypt(it, oldKey) }
                )
            }
            
            // 4. Configurar nuevo password maestro
            authRepository.setupMasterPassword(newPassword)
            
            // 5. Re-cifrar todas las entradas con nueva clave
            val newKey = authRepository.getMasterKey()!!
            decryptedEntries.forEach { entry ->
                passwordRepository.updateEntry(
                    entry.copy(
                        password = encrypt(entry.password, newKey),
                        notes = entry.notes?.let { encrypt(it, newKey) }
                    )
                )
            }
            
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

## Casos de Uso - Gestión de Contraseñas

### GetAllPasswords

Obtiene todas las contraseñas del usuario.

```kotlin
class GetAllPasswords(private val repository: PasswordRepository) {
    suspend operator fun invoke(): Flow<List<PasswordEntry>> {
        return repository.getAllEntries()
    }
}
```

**Características**:
- Devuelve Flow para reactividad
- Las contraseñas vienen descifradas
- Emite automáticamente cuando hay cambios en BD

---

### GetPasswordById

Obtiene una contraseña específica por ID.

```kotlin
class GetPasswordById(private val repository: PasswordRepository) {
    suspend operator fun invoke(id: String): PasswordEntry? {
        return repository.getEntryById(id)
    }
}
```

---

### CreatePasswordEntry

Crea una nueva entrada de contraseña.

```kotlin
class CreatePasswordEntry(private val repository: PasswordRepository) {
    suspend operator fun invoke(entry: PasswordEntry): Result<Unit> {
        return try {
            // Validaciones
            if (entry.title.isBlank()) {
                return Result.failure(EmptyTitleException())
            }
            
            if (entry.password.isBlank()) {
                return Result.failure(EmptyPasswordException())
            }
            
            repository.insertEntry(entry)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

### UpdatePasswordEntry

Actualiza una contraseña existente.

```kotlin
class UpdatePasswordEntry(private val repository: PasswordRepository) {
    suspend operator fun invoke(entry: PasswordEntry): Result<Unit> {
        return try {
            repository.updateEntry(entry)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

### DeletePasswordEntry

Elimina una contraseña.

```kotlin
class DeletePasswordEntry(private val repository: PasswordRepository) {
    suspend operator fun invoke(entry: PasswordEntry): Result<Unit> {
        return try {
            repository.deleteEntry(entry)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

### SearchPasswords

Busca contraseñas por título, username o URL.

```kotlin
class SearchPasswords(private val repository: PasswordRepository) {
    suspend operator fun invoke(query: String): Flow<List<PasswordEntry>> {
        if (query.isBlank()) {
            return repository.getAllEntries()
        }
        return repository.searchEntries("%$query%")
    }
}
```

**Criterios de búsqueda**:
- Búsqueda case-insensitive
- Busca en: título, username, URL
- Wildcards automáticos (%query%)

---

### ToggleFavoriteEntry

Marca/desmarca una contraseña como favorita.

```kotlin
class ToggleFavoriteEntry(private val repository: PasswordRepository) {
    suspend operator fun invoke(entry: PasswordEntry): Result<Unit> {
        return try {
            val updatedEntry = entry.copy(isFavorite = !entry.isFavorite)
            repository.updateEntry(updatedEntry)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

### GeneratePassword

Genera una contraseña segura aleatoria.

```kotlin
class GeneratePassword {
    suspend operator fun invoke(options: PasswordOptions): String {
        val charPool = buildCharPool(options)
        return (1..options.length)
            .map { SecureRandom().nextInt(charPool.size) }
            .map(charPool::get)
            .joinToString("")
    }
    
    private fun buildCharPool(options: PasswordOptions): String {
        return buildString {
            if (options.includeUppercase) append('A'..'Z')
            if (options.includeLowercase) append('a'..'z')
            if (options.includeNumbers) append('0'..'9')
            if (options.includeSymbols) append("!@#$%^&*()_+-=[]{}|;:,.<>?")
        }
    }
}

data class PasswordOptions(
    val length: Int = 16,
    val includeUppercase: Boolean = true,
    val includeLowercase: Boolean = true,
    val includeNumbers: Boolean = true,
    val includeSymbols: Boolean = true
)
```

**Opciones configurables**:
- Longitud (8-128 caracteres)
- Mayúsculas
- Minúsculas
- Números
- Símbolos especiales

---

## Casos de Uso - Gestión de Categorías

### GetCategories

Obtiene todas las categorías disponibles.

```kotlin
class GetCategories(private val repository: CategoryRepository) {
    suspend operator fun invoke(): Flow<List<Category>> {
        return repository.getAllCategories()
    }
}
```

---

### CreateCategory

Crea una nueva categoría personalizada.

```kotlin
class CreateCategory(private val repository: CategoryRepository) {
    suspend operator fun invoke(category: Category): Result<Unit> {
        return try {
            if (category.name.isBlank()) {
                return Result.failure(EmptyCategoryNameException())
            }
            
            // Las categorías nuevas son siempre custom y deletable
            val newCategory = category.copy(
                isCustom = true,
                isDeletable = true
            )
            
            repository.createCategory(newCategory)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

### UpdateCategory

Actualiza una categoría existente.

```kotlin
class UpdateCategory(private val repository: CategoryRepository) {
    suspend operator fun invoke(category: Category): Result<Unit> {
        return try {
            // No permitir cambiar isDeletable de predefinidas
            if (!category.isDeletable) {
                return Result.failure(CannotModifySystemCategoryException())
            }
            
            repository.updateCategory(category)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

### DeleteCategory

Elimina una categoría personalizada.

```kotlin
class DeleteCategory(
    private val repository: CategoryRepository,
    private val passwordRepository: PasswordRepository
) {
    suspend operator fun invoke(category: Category): Result<Unit> {
        return try {
            // No permitir eliminar categorías predefinidas
            if (!category.isDeletable) {
                return Result.failure(CannotDeleteSystemCategoryException())
            }
            
            // Mover contraseñas a "General" antes de eliminar
            val generalCategory = getAllCategories().first { it.name == "General" }
            val entries = passwordRepository.getEntriesByCategory(category.id).first()
            
            entries.forEach { entry ->
                passwordRepository.updateEntry(entry.copy(categoryId = generalCategory.id))
            }
            
            repository.deleteCategory(category)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

### SeedPredefinedCategories

Inicializa categorías predefinidas en primera ejecución.

```kotlin
class SeedPredefinedCategories(private val categoryDao: CategoryDao) {
    suspend operator fun invoke() {
        if (needsSeed(categoryDao)) {
            seed(categoryDao)
        }
    }
    
    private suspend fun needsSeed(dao: CategoryDao): Boolean {
        val count = dao.getCount()
        return count == 0
    }
    
    private suspend fun seed(dao: CategoryDao) {
        val predefinedCategories = listOf(
            Category(id = UUID.randomUUID().toString(), name = "General", color = 0xFF9E9E9E.toInt(), icon = "FaFolder", isCustom = false, isDeletable = false),
            Category(id = UUID.randomUUID().toString(), name = "Redes Sociales", color = 0xFF2196F3.toInt(), icon = "FaUsers", isCustom = false, isDeletable = false),
            Category(id = UUID.randomUUID().toString(), name = "Finanzas", color = 0xFF4CAF50.toInt(), icon = "FaMoneyBill", isCustom = false, isDeletable = false),
            Category(id = UUID.randomUUID().toString(), name = "Compras", color = 0xFFFF9800.toInt(), icon = "FaShoppingCart", isCustom = false, isDeletable = false),
            Category(id = UUID.randomUUID().toString(), name = "Entretenimiento", color = 0xFFF44336.toInt(), icon = "FaFilm", isCustom = false, isDeletable = false),
            Category(id = UUID.randomUUID().toString(), name = "Trabajo", color = 0xFF9C27B0.toInt(), icon = "FaBriefcase", isCustom = false, isDeletable = false),
            Category(id = UUID.randomUUID().toString(), name = "Educación", color = 0xFFFFEB3B.toInt(), icon = "FaGraduationCap", isCustom = false, isDeletable = false),
            Category(id = UUID.randomUUID().toString(), name = "Salud", color = 0xFFFFC0CB.toInt(), icon = "FaHeartbeat", isCustom = false, isDeletable = false),
            Category(id = UUID.randomUUID().toString(), name = "Viajes", color = 0xFF00BCD4.toInt(), icon = "FaPlane", isCustom = false, isDeletable = false),
            Category(id = UUID.randomUUID().toString(), name = "Otros", color = 0xFF795548.toInt(), icon = "FaArchive", isCustom = false, isDeletable = false)
        )
        
        dao.insertAll(predefinedCategories.map { it.toEntity() })
    }
}
```

---

## Casos de Uso - Configuración

### GetSettings / UpdateSettings

```kotlin
class GetSettings(private val repository: SettingsRepository) {
    suspend operator fun invoke(): AppSettings {
        return repository.getSettings()
    }
}

class UpdateSettings(private val repository: SettingsRepository) {
    suspend operator fun invoke(settings: AppSettings): Result<Unit> {
        return try {
            repository.updateSettings(settings)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### Casos de Uso Individuales

Para configuración granular:

```kotlin
class GetThemeMode(private val repository: SettingsRepository) {
    suspend operator fun invoke(): ThemeMode {
        return repository.getThemeMode()
    }
}

class SetThemeMode(private val repository: SettingsRepository) {
    suspend operator fun invoke(mode: ThemeMode): Result<Unit> {
        return try {
            repository.setThemeMode(mode)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

class IsBiometricEnabled(private val repository: SettingsRepository) {
    suspend operator fun invoke(): Boolean {
        return repository.isBiometricEnabled()
    }
}

class SetBiometricEnabled(private val repository: SettingsRepository) {
    suspend operator fun invoke(enabled: Boolean): Result<Unit> {
        return try {
            repository.setBiometricEnabled(enabled)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

class GetLockTimeout(private val repository: SettingsRepository) {
    suspend operator fun invoke(): Long {
        return repository.getLockTimeout()
    }
}

class SetLockTimeout(private val repository: SettingsRepository) {
    suspend operator fun invoke(timeoutMs: Long): Result<Unit> {
        return try {
            repository.setLockTimeout(timeoutMs)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

## Casos de Uso - Backup

### ExportEncryptedBackup

Exporta todos los datos en un archivo cifrado.

```kotlin
class ExportEncryptedBackup(
    private val passwordRepository: PasswordRepository,
    private val categoryRepository: CategoryRepository,
    private val dataCipher: DataCipher,
    private val authRepository: AuthRepository,
    private val context: Context
) {
    suspend operator fun invoke(outputPath: String): Result<Unit> {
        return try {
            // 1. Obtener todos los datos
            val entries = passwordRepository.getAllEntriesList()
            val categories = categoryRepository.getAllCategories().first()
            
            // 2. Cifrar datos con clave de backup
            val masterKey = authRepository.getMasterKey()!!
            val backupKey = deriveBackupKey(masterKey)
            
            val encryptedData = BackupData(
                entries = entries.map { encryptEntry(it, backupKey) },
                categories = categories
            )
            
            // 3. Serializar y guardar
            val json = Json.encodeToString(encryptedData)
            val file = File(outputPath)
            file.writeText(json)
            
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

### ImportFromCSV

Importa contraseñas desde archivo CSV.

```kotlin
class ImportFromCSV(
    private val passwordRepository: PasswordRepository,
    private val context: Context
) {
    suspend operator fun invoke(inputPath: String): Result<Int> {
        return try {
            val file = File(inputPath)
            val lines = file.readLines()
            
            var importedCount = 0
            for ((index, line) in lines.withIndex()) {
                if (index == 0) continue  // Skip header
                
                val columns = line.split(",")
                if (columns.size >= 3) {
                    val entry = PasswordEntry(
                        id = UUID.randomUUID().toString(),
                        title = columns[0].trim(),
                        username = columns[1].trim(),
                        password = columns[2].trim(),
                        url = columns.getOrNull(3)?.trim(),
                        notes = null,
                        categoryId = getDefaultCategoryId(),
                        icon = null,
                        isFavorite = false,
                        createdAt = System.currentTimeMillis(),
                        updatedAt = System.currentTimeMillis()
                    )
                    
                    passwordRepository.insertEntry(entry)
                    importedCount++
                }
            }
            
            Result.success(importedCount)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

## Casos de Uso - Auditoría y Estadísticas

### AuditWeakPasswords

Detecta contraseñas débiles en la base de datos.

```kotlin
class AuditWeakPasswords(
    private val passwordRepository: PasswordRepository
) {
    suspend operator fun invoke(): AuditResult {
        val entries = passwordRepository.getAllEntriesList()
        
        val weakPasswords = entries.filter { entry ->
            !PasswordValidator.isStrong(entry.password)
        }.map { entry ->
            WeakPasswordEntry(
                id = entry.id,
                title = entry.title,
                password = entry.password,
                weaknessReason = getWeaknessReason(entry.password),
                strengthScore = calculateStrength(entry.password)
            )
        }
        
        return AuditResult(
            totalEntries = entries.size,
            weakCount = weakPasswords.size,
            weakEntries = weakPasswords
        )
    }
}
```

---

### GetSecurityStatistics

Obtiene estadísticas de seguridad.

```kotlin
class GetSecurityStatistics(
    private val passwordRepository: PasswordRepository,
    private val categoryRepository: CategoryRepository
) {
    suspend operator fun invoke(): SecurityStatistics {
        val entries = passwordRepository.getAllEntriesList()
        val categories = categoryRepository.getAllCategories().first()
        
        return SecurityStatistics(
            totalPasswords = entries.size,
            favoriteCount = entries.count { it.isFavorite },
            categoryCount = categories.size,
            weakPasswordCount = entries.count { !PasswordValidator.isStrong(it.password) },
            reusedPasswordCount = entries.groupBy { it.password }.filter { it.value.size > 1 }.size,
            oldPasswordCount = entries.count { it.isOld() },
            averageStrength = entries.map { calculateStrength(it.password) }.average()
        )
    }
}
```
