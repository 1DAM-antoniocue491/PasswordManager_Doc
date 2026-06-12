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
    │   ├── UpdatePasswordEntry.kt
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

## Modelos de Dominio

### PasswordEntry

Modelo principal que representa una entrada de contraseña.

```kotlin
data class PasswordEntry(
    val id: String,
    val title: String,
    val username: String,
    val password: String,
    val url: String?,
    val notes: String?,
    val categoryId: String,
    val icon: String?,
    val isFavorite: Boolean,
    val createdAt: Long,
    val updatedAt: Long
)
```

**Reglas de negocio**:
- `id`: UUID generado al crear
- `password`: Siempre cifrado cuando se persiste
- `notes`: Opcional, cifrado cuando existe
- `isFavorite`: Determina si aparece en sección de favoritos
- `createdAt/updatedAt`: Timestamps en milisegundos epoch

### Category

Modelo para categorías de organización.

```kotlin
@Serializable
data class Category(
    val id: String,
    val name: String,
    val color: Int,
    val icon: String,
    val isCustom: Boolean,
    val isDeletable: Boolean
)
```

**Tipos de categorías**:

| Tipo | isCustom | isDeletable | Ejemplo |
|------|----------|-------------|---------|
| Predefinida | false | false | General, Finanzas |
| Personalizada | true | true | Categorías creadas por usuario |

### AppSettings

Configuración de la aplicación.

```kotlin
data class AppSettings(
    val themeMode: ThemeMode = ThemeMode.AUTO,
    val isBiometricEnabled: Boolean = false,
    val lockTimeoutMs: Long = 300_000,  // 5 minutos
    val hasMasterPassword: Boolean = false
)

enum class ThemeMode(val value: Int) {
    AUTO(0),
    LIGHT(1),
    DARK(2)
}
```

### BackupData

Estructura para exportar/importar datos.

```kotlin
data class BackupData(
    val version: String = "1.0",
    val exportDate: Long = System.currentTimeMillis(),
    val entries: List<EncryptedPasswordEntry>,
    val categories: List<Category>
)

@Serializable
data class EncryptedPasswordEntry(
    val id: String,
    val title: String,
    val username: String,
    val encryptedPassword: String,  // Base64
    val url: String?,
    val encryptedNotes: String?,    // Base64
    val categoryId: String,
    val icon: String?,
    val isFavorite: Boolean,
    val createdAt: Long,
    val updatedAt: Long
)
```

## Interfaces de Repositorio

### PasswordRepository

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

### AuthRepository

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
    fun clearMasterKey()
}
```

### CategoryRepository

```kotlin
interface CategoryRepository {
    fun getAllCategories(): Flow<List<Category>>
    suspend fun createCategory(category: Category)
    suspend fun updateCategory(category: Category)
    suspend fun deleteCategory(category: Category)
}
```

### SettingsRepository

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

## Casos de Uso

### Patrón de Diseño

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

### GetPasswordById

Obtiene una contraseña específica por ID.

```kotlin
class GetPasswordById(private val repository: PasswordRepository) {
    suspend operator fun invoke(id: String): PasswordEntry? {
        return repository.getEntryById(id)
    }
}
```

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

### UpdatePasswordEntry

Actualiza una contraseña existente.

```kotlin
class UpdatePasswordEntry(private val repository: PasswordRepository) {
    suspend operator fun invoke(entry: PasswordEntry): Result<Unit> {
        return try {
            // El repositorio actualiza el timestamp automáticamente
            repository.updateEntry(entry)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

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

Obtiene y actualiza configuración de la aplicación.

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
            
            // 2. Cifrar datos con clave de backup (derivada del master key)
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
                reason = PasswordValidator.getWeaknessReason(entry.password)
            )
        }
        
        return AuditResult(
            totalEntries = entries.size,
            weakCount = weakPasswords.size,
            weakEntries = weakPasswords
        )
    }
}

data class AuditResult(
    val totalEntries: Int,
    val weakCount: Int,
    val weakEntries: List<WeakPasswordEntry>
)

data class WeakPasswordEntry(
    val id: String,
    val title: String,
    val reason: String
)
```

**Criterios de debilidad**:
- Menos de 8 caracteres
- Sin mayúsculas
- Sin números
- Sin símbolos
- Patrones comunes (123456, password, qwerty)

### GetSecurityStatistics

Obtiene estadísticas de seguridad del usuario.

```kotlin
class GetSecurityStatistics(
    private val passwordRepository: PasswordRepository,
    private val categoryRepository: CategoryRepository,
    private val auditWeakPasswords: AuditWeakPasswords
) {
    suspend operator fun invoke(): SecurityStatistics {
        val entries = passwordRepository.getAllEntriesList()
        val categories = categoryRepository.getAllCategories().first()
        val audit = auditWeakPasswords()
        
        // Calcular fortaleza promedio
        val strengthScores = entries.map { entry ->
            PasswordValidator.getStrengthScore(entry.password)
        }
        val averageStrength = strengthScores.average()
        
        // Contar contraseñas reutilizadas (mismo password)
        val passwordGroups = entries.groupBy { it.password }
        val reusedCount = passwordGroups.count { it.value.size > 1 }
        
        return SecurityStatistics(
            totalPasswords = entries.size,
            favoriteCount = entries.count { it.isFavorite },
            categoryCount = categories.size,
            weakPasswordCount = audit.weakCount,
            reusedPasswordCount = reusedCount,
            averageStrength = averageStrength,
            lastUpdated = System.currentTimeMillis()
        )
    }
}

data class SecurityStatistics(
    val totalPasswords: Int,
    val favoriteCount: Int,
    val categoryCount: Int,
    val weakPasswordCount: Int,
    val reusedPasswordCount: Int,
    val averageStrength: Double,  // 0.0 - 100.0
    val lastUpdated: Long
)
```

---

## Testing de la Capa de Dominio

### Unit Tests para Casos de Uso

```kotlin
class CreatePasswordEntryTest {
    
    private val repository = MockkPasswordRepository()
    private val useCase = CreatePasswordEntry(repository)
    
    @Test
    fun `should create entry when data is valid`() = runTest {
        // Given
        val entry = PasswordEntry(
            id = "test-id",
            title = "Test Entry",
            username = "user@example.com",
            password = "SecurePass123!",
            url = "https://example.com",
            notes = null,
            categoryId = "cat-1",
            icon = null,
            isFavorite = false,
            createdAt = System.currentTimeMillis(),
            updatedAt = System.currentTimeMillis()
        )
        
        coEvery { repository.insertEntry(entry) } returns Unit
        
        // When
        val result = useCase(entry)
        
        // Then
        assertTrue(result.isSuccess)
        coVerify { repository.insertEntry(entry) }
    }
    
    @Test
    fun `should fail when title is empty`() = runTest {
        // Given
        val entry = PasswordEntry(
            id = "test-id",
            title = "",  // Empty title
            username = "user",
            password = "SecurePass123!",
            url = null,
            notes = null,
            categoryId = "cat-1",
            icon = null,
            isFavorite = false,
            createdAt = System.currentTimeMillis(),
            updatedAt = System.currentTimeMillis()
        )
        
        // When
        val result = useCase(entry)
        
        // Then
        assertTrue(result.isFailure)
        assertTrue(result.exceptionOrNull() is EmptyTitleException)
    }
}
```

### Tests para Validadores

```kotlin
class PasswordValidatorTest {
    
    @Test
    fun `strong password should pass validation`() {
        // Given
        val password = "SecureP@ssw0rd123!"
        
        // When
        val isValid = PasswordValidator.isValid(password.toCharArray())
        
        // Then
        assertTrue(isValid)
    }
    
    @Test
    fun `short password should fail validation`() {
        // Given
        val password = "Short1!"
        
        // When
        val isValid = PasswordValidator.isValid(password.toCharArray())
        
        // Then
        assertFalse(isValid)
    }
    
    @Test
    fun `password without uppercase should fail validation`() {
        // Given
        val password = "lowercase123!"
        
        // When
        val isValid = PasswordValidator.isValid(password.toCharArray())
        
        // Then
        assertFalse(isValid)
    }
}
```

---

**Documentación Relacionada:**
- [Arquitectura](../arquitectura/overview.md)
- [Capa de Datos](../data/overview.md)
- [Capa de Presentación](../presentation/overview.md)
