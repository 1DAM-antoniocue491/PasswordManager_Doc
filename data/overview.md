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

## Base de Datos Room

### PasswordDatabase

La base de datos principal contiene tres tablas:

```kotlin
@Database(
    entities = [
        PasswordEntryEntity::class,
        CategoryEntity::class,
        SettingsEntity::class
    ],
    version = 1,
    exportSchema = false
)
@TypeConverters(DateConverters::class)
abstract class PasswordDatabase : RoomDatabase() {
    abstract fun passwordEntryDao(): PasswordEntryDao
    abstract fun categoryDao(): CategoryDao
    abstract fun settingsDao(): SettingsDao
    
    companion object {
        fun getInstance(context: Context): PasswordDatabase {
            return INSTANCE ?: synchronized(this) {
                Room.databaseBuilder(
                    context.applicationContext,
                    PasswordDatabase::class.java,
                    "passwords.db"
                ).build()
            }
        }
    }
}
```

**Características**:
- **Nombre**: `passwords.db`
- **Versión**: 1
- **TipoConverters**: `DateConverters` para manejar timestamps

## Entidades

### PasswordEntryEntity

Representa una entrada de contraseña en la base de datos.

```kotlin
@Entity(tableName = "password_entries")
data class PasswordEntryEntity(
    @PrimaryKey val id: String,
    val title: String,
    val username: String,
    val password: String,        // Cifrado (Base64)
    val url: String?,
    val notes: String?,          // Cifrado (Base64)
    val categoryId: String,      // FK a categories
    val icon: String?,
    val isFavorite: Boolean,
    val createdAt: Long,         // Timestamp epoch
    val updatedAt: Long          // Timestamp epoch
)
```

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | String | UUID único, Primary Key |
| `title` | String | Título descriptivo |
| `username` | String | Nombre de usuario |
| `password` | String | Contraseña cifrada en Base64 |
| `url` | String? | URL del sitio (opcional) |
| `notes` | String? | Notas adicionales cifradas |
| `categoryId` | String | FK a `categories(id)` |
| `icon` | String? | Nombre del icono (ej: `FaLock`) |
| `isFavorite` | Boolean | Marcado como favorito |
| `createdAt` | Long | Timestamp de creación (epoch ms) |
| `updatedAt` | Long | Timestamp de actualización (epoch ms) |

**Índices implícitos**:
- Primary Key en `id`
- Foreign Key implícita en `categoryId`

### CategoryEntity

Representa una categoría para organizar contraseñas.

```kotlin
@Entity(tableName = "categories")
data class CategoryEntity(
    @PrimaryKey val id: String,
    val name: String,
    val color: Int,              // Color ARGB
    val icon: String,
    val isCustom: Boolean,
    val isDeletable: Boolean
)
```

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | String | UUID único |
| `name` | String | Nombre de la categoría |
| `color` | Int | Color ARGB (ej: 0xFFFF5722) |
| `icon` | String | Icono de Font Awesome |
| `isCustom` | Boolean | true = creada por usuario |
| `isDeletable` | Boolean | false = predefinida (no se puede eliminar) |

**Categorías Predefinidas**:
- General (gris)
- Redes Sociales (azul)
- Finanzas (verde)
- Compras (naranja)
- Entretenimiento (rojo)
- Trabajo (morado)
- Educación (amarillo)
- Salud (rosa)
- Viajes (cyan)
- Otros (marrón)

### SettingsEntity

Almacena configuración clave-valor.

```kotlin
@Entity(tableName = "settings")
data class SettingsEntity(
    @PrimaryKey val key: String,
    val value: String,
    val updatedAt: Long
)
```

**Claves utilizadas**:
| Key | Value Type | Descripción |
|-----|------------|-------------|
| `theme_mode` | Int | 0=Auto, 1=Light, 2=Dark |
| `biometric_enabled` | Boolean | Si biometría está activa |
| `lock_timeout` | Long | Timeout en milisegundos |

## Data Access Objects (DAO)

### PasswordEntryDao

Operaciones CRUD para entradas de contraseñas.

```kotlin
@Dao
interface PasswordEntryDao {
    
    // Lectura reactiva con Flow
    @Query("SELECT * FROM password_entries ORDER BY title ASC")
    fun getAll(): Flow<List<PasswordEntryEntity>>
    
    @Query("SELECT * FROM password_entries WHERE id = :id")
    suspend fun getById(id: String): PasswordEntryEntity?
    
    @Query("SELECT * FROM password_entries WHERE title LIKE :query OR username LIKE :query OR url LIKE :query")
    fun search(query: String): Flow<List<PasswordEntryEntity>>
    
    @Query("SELECT * FROM password_entries WHERE categoryId = :categoryId ORDER BY title ASC")
    fun getByCategory(categoryId: String): Flow<List<PasswordEntryEntity>>
    
    @Query("SELECT * FROM password_entries WHERE isFavorite = 1 ORDER BY title ASC")
    fun getFavorites(): Flow<List<PasswordEntryEntity>>
    
    // Inserción/Actualización
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(entry: PasswordEntryEntity)
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(entries: List<PasswordEntryEntity>)
    
    // Eliminación
    @Delete
    suspend fun delete(entry: PasswordEntryEntity)
    
    @Query("DELETE FROM password_entries WHERE id = :id")
    suspend fun deleteById(id: String)
    
    @Query("DELETE FROM password_entries WHERE categoryId = :categoryId")
    suspend fun deleteByCategory(categoryId: String)
    
    // Conteos
    @Query("SELECT COUNT(*) FROM password_entries")
    suspend fun getCount(): Int
    
    @Query("SELECT COUNT(*) FROM password_entries WHERE categoryId = :categoryId")
    suspend fun getCountByCategory(categoryId: String): Int
}
```

**Características**:
- **Flow**: Todas las lecturas devuelven Flow para reactividad
- **REPLACE**: En inserción, reemplaza si existe (por PK)
- **Cascada**: `deleteByCategory` elimina entradas al eliminar categoría

### CategoryDao

```kotlin
@Dao
interface CategoryDao {
    @Query("SELECT * FROM categories ORDER BY name ASC")
    fun getAll(): Flow<List<CategoryEntity>>
    
    @Query("SELECT * FROM categories WHERE id = :id")
    suspend fun getById(id: String): CategoryEntity?
    
    @Query("SELECT * FROM categories WHERE isCustom = 1 ORDER BY name ASC")
    fun getCustomCategories(): Flow<List<CategoryEntity>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(category: CategoryEntity)
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(categories: List<CategoryEntity>)
    
    @Delete
    suspend fun delete(category: CategoryEntity)
    
    @Query("DELETE FROM categories WHERE id = :id")
    suspend fun deleteById(id: String)
}
```

### SettingsDao

```kotlin
@Dao
interface SettingsDao {
    @Query("SELECT * FROM settings")
    fun getAll(): Flow<List<SettingsEntity>>
    
    @Query("SELECT * FROM settings WHERE key = :key")
    suspend fun getByKey(key: String): SettingsEntity?
    
    @Query("SELECT * FROM settings WHERE key = :key")
    fun getByKeyFlow(key: String): Flow<SettingsEntity?>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(setting: SettingsEntity)
    
    @Query("DELETE FROM settings WHERE key = :key")
    suspend fun deleteByKey(key: String)
}
```

## TypeConverters

### DateConverters

Convierte entre `Long` (timestamp) y `Date` para Room.

```kotlin
class DateConverters {
    @TypeConverter
    fun fromTimestamp(value: Long): Date {
        return Date(value)
    }
    
    @TypeConverter
    fun dateToTimestamp(date: Date): Long {
        return date.time
    }
}
```

**Nota**: Las entidades usan `Long` directamente, pero este converter permite usar `Date` si es necesario.

## Repositorios (Implementaciones)

### PasswordRepositoryImpl

Implementa `PasswordRepository` de la capa de dominio.

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
    
    override suspend fun insertEntry(entry: PasswordEntry) {
        val masterKey = getMasterKeyOrThrow()
        val encryptedEntry = encryptEntry(entry, masterKey)
        passwordEntryDao.insert(encryptedEntry)
    }
    
    private fun decryptEntry(entity: PasswordEntryEntity): PasswordEntry? {
        return try {
            val masterKey = authRepository.getMasterKey() ?: return null
            
            // Decodificar Base64 y descifrar
            val decryptedPassword = try {
                val decoded = entity.password.fromBase64String()
                dataCipher.decrypt(decoded, masterKey)
            } catch (e: Exception) {
                entity.password  // Fallback si falla
            }
            
            val decryptedNotes = entity.notes?.let {
                try {
                    val decoded = it.fromBase64String()
                    dataCipher.decrypt(decoded, masterKey)
                } catch (e: Exception) {
                    it
                }
            }
            
            PasswordEntry(
                id = entity.id,
                title = entity.title,
                username = entity.username,
                password = decryptedPassword,
                url = entity.url,
                notes = decryptedNotes,
                categoryId = entity.categoryId,
                icon = entity.icon,
                isFavorite = entity.isFavorite,
                createdAt = entity.createdAt,
                updatedAt = entity.updatedAt
            )
        } catch (e: Exception) {
            null
        }
    }
    
    private fun encryptEntry(entry: PasswordEntry, masterKey: ByteArray): PasswordEntryEntity {
        val encryptedPassword = dataCipher.encrypt(entry.password, masterKey)
            .toBase64String()
        
        val encryptedNotes = entry.notes?.let {
            dataCipher.encrypt(it, masterKey).toBase64String()
        }
        
        return PasswordEntryEntity(
            id = entry.id,
            title = entry.title,
            username = entry.username,
            password = encryptedPassword,
            url = entry.url,
            notes = encryptedNotes,
            categoryId = entry.categoryId,
            icon = entry.icon,
            isFavorite = entry.isFavorite,
            createdAt = entry.createdAt,
            updatedAt = System.currentTimeMillis()
        )
    }
}
```

**Responsabilidades**:
1. **Cifrado**: Cifra password y notes antes de guardar en BD
2. **Descifrado**: Descifra al leer de BD
3. **Base64**: Codifica datos cifrados para almacenamiento
4. **Validación**: Verifica sesión activa antes de operar

### AuthRepositoryImpl

Gestiona autenticación y clave maestra.

```kotlin
class AuthRepositoryImpl(
    private val keystoreManager: KeystoreManager,
    private val passwordDeriver: PasswordDeriver,
    private val cipherManager: CipherManager,
    private val dataCipher: DataCipher,
    private val secureStorage: SecureStorage
) : AuthRepository {
    
    override suspend fun setupMasterPassword(password: CharArray): Result<Unit> {
        return try {
            // 1. Generar salt
            val salt = passwordDeriver.generateSalt()
            
            // 2. Derivar clave del password
            val derivedKey = passwordDeriver.deriveKey(password, salt)
            
            // 3. Generar claves RSA en Keystore
            keystoreManager.generateKeyIfNeeded()
            val publicKey = keystoreManager.getPublicKey()
            
            // 4. Cifrar clave derivada con RSA
            val encryptedKey = cipherManager.encrypt(derivedKey, publicKey)
            
            // 5. Guardar en SecureStorage
            secureStorage.saveSalt(salt)
            secureStorage.saveEncryptedKey(encryptedKey)
            
            // 6. Seed de categorías predefinidas
            CategorySeed.seed(categoryDao)
            
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override suspend fun authenticateUser(password: CharArray): Result<ByteArray> {
        return try {
            // 1. Leer salt y clave cifrada
            val salt = secureStorage.getSalt() 
                ?: return Result.failure(SecurityException("No hay configuración"))
            
            val encryptedKey = secureStorage.getEncryptedKey()
                ?: return Result.failure(SecurityException("No hay clave almacenada"))
            
            // 2. Derivar clave del password ingresado
            val candidateKey = passwordDeriver.deriveKey(password, salt)
            
            // 3. Descifrar clave almacenada (requiere biometría)
            val privateKey = keystoreManager.getPrivateKey()
            val storedKey = cipherManager.decrypt(encryptedKey, privateKey)
            
            // 4. Comparar claves
            if (!candidateKey.contentEquals(storedKey)) {
                return Result.failure(SecurityException("Password incorrecto"))
            }
            
            // 5. Autenticación exitosa - retornar clave para sesión
            Result.success(storedKey)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### CategoryRepositoryImpl

```kotlin
class CategoryRepositoryImpl(
    private val categoryDao: CategoryDao
) : CategoryRepository {
    
    override fun getAllCategories(): Flow<List<Category>> {
        return categoryDao.getAll().map { entities ->
            entities.map { it.toDomain() }
        }
    }
    
    override suspend fun createCategory(category: Category) {
        categoryDao.insert(category.toEntity())
    }
    
    override suspend fun updateCategory(category: Category) {
        categoryDao.insert(category.toEntity())  // REPLACE por PK
    }
    
    override suspend fun deleteCategory(category: Category) {
        categoryDao.delete(category.toEntity())
    }
}
```

### SettingsRepositoryImpl

Usa DataStore para preferencias.

```kotlin
class SettingsRepositoryImpl(
    private val dataStore: DataStore<Preferences>
) : SettingsRepository {
    
    override suspend fun getThemeMode(): ThemeMode {
        return dataStore.data.first()[PREFERENCES_KEY_THEME_MODE]?.let {
            ThemeMode.fromInt(it)
        } ?: ThemeMode.AUTO
    }
    
    override suspend fun setThemeMode(mode: ThemeMode) {
        dataStore.edit { preferences ->
            preferences[PREFERENCES_KEY_THEME_MODE] = mode.value
        }
    }
    
    override suspend fun isBiometricEnabled(): Boolean {
        return dataStore.data.first()[PREFERENCES_KEY_BIOMETRIC] ?: false
    }
    
    override suspend fun setBiometricEnabled(enabled: Boolean) {
        dataStore.edit { preferences ->
            preferences[PREFERENCES_KEY_BIOMETRIC] = enabled
        }
    }
}
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

## Manejo de Transacciones

Room maneja transacciones automáticamente para operaciones individuales. Para operaciones múltiples atómicas:

```kotlin
@Transaction
suspend fun migrateCategory(oldCategoryId: String, newCategoryId: String) {
    // 1. Obtener todas las entradas de la categoría vieja
    val entries = passwordEntryDao.getByCategory(oldCategoryId).first()
    
    // 2. Actualizar cada entrada a la nueva categoría
    entries.forEach { entry ->
        passwordEntryDao.insert(entry.copy(categoryId = newCategoryId))
    }
    
    // 3. Eliminar categoría vieja
    categoryDao.deleteById(oldCategoryId)
}
```

## Migraciones de Base de Datos

Actualmente la base de datos está en versión 1. Para futuras migraciones:

```kotlin
Room.databaseBuilder(
    context,
    PasswordDatabase::class.java,
    "passwords.db"
)
.addMigrations(object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // Agregar nueva columna
        database.execSQL("ALTER TABLE password_entries ADD COLUMN lastUsedAt INTEGER")
    }
})
.build()
```

## Testing de la Capa de Datos

### Unit Tests para DAO

```kotlin
@RunWith(AndroidJUnit4::class)
class PasswordEntryDaoTest {
    
    private lateinit var database: PasswordDatabase
    private lateinit var dao: PasswordEntryDao
    
    @Before
    fun setup() {
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            PasswordDatabase::class.java
        ).build()
        dao = database.passwordEntryDao()
    }
    
    @After
    fun teardown() {
        database.close()
    }
    
    @Test
    fun insertAndGetEntry() = runTest {
        // Given
        val entry = PasswordEntryEntity(
            id = "test-id",
            title = "Test",
            username = "user",
            password = "encrypted-pass",
            url = null,
            notes = null,
            categoryId = "cat-1",
            icon = null,
            isFavorite = false,
            createdAt = System.currentTimeMillis(),
            updatedAt = System.currentTimeMillis()
        )
        
        // When
        dao.insert(entry)
        val result = dao.getById("test-id")
        
        // Then
        assertEquals(entry.id, result?.id)
        assertEquals(entry.title, result?.title)
    }
}
```

## Consideraciones de Rendimiento

### Índices

Para mejorar rendimiento en búsquedas:

```kotlin
@Entity(
    tableName = "password_entries",
    indices = [
        Index(value = ["title"]),
        Index(value = ["categoryId"]),
        Index(value = ["isFavorite"])
    ]
)
data class PasswordEntryEntity(...)
```

### Consultas Optimizadas

```kotlin
// ✅ BUENO: Usa índice en categoryId
@Query("SELECT * FROM password_entries WHERE categoryId = :categoryId")
fun getByCategory(categoryId: String): Flow<List<PasswordEntryEntity>>

// ❌ MALO: No usa índice, escaneo completo
@Query("SELECT * FROM password_entries WHERE title LIKE '%' || :query || '%'")
fun searchSlow(query: String): Flow<List<PasswordEntryEntity>>

// ✅ MEJOR: LIKE con wildcard al final usa índice
@Query("SELECT * FROM password_entries WHERE title LIKE :query || '%'")
fun searchOptimized(query: String): Flow<List<PasswordEntryEntity>>
```

## Componentes de Seguridad (Implementación)

### PasswordDeriver

**Archivo**: `security/PasswordDeriver.kt`

Deriva clave simétrica del password maestro usando PBKDF2-HMAC-SHA256.

```kotlin
class PasswordDeriver {
    fun generateSalt(): ByteArray {
        val salt = ByteArray(SALT_SIZE)
        secureRandom.nextBytes(salt)
        return salt
    }
    
    fun deriveKey(password: CharArray, salt: ByteArray): ByteArray {
        val keySpec = PBEKeySpec(
            password,
            salt,
            PBKDF2_ITERATIONS,  // 100,000
            KEY_LENGTH_BITS     // 256
        )
        
        val secretKeyFactory = SecretKeyFactory.getInstance(PBKDF2_ALGORITHM)
        val keyBytes = secretKeyFactory.generateSecret(keySpec).encoded
        
        keySpec.clearPassword()  // Limpiar memoria
        password.fillPassword()   // Limpiar password
        
        return keyBytes
    }
    
    companion object {
        private const val PBKDF2_ALGORITHM = "PBKDF2withHmacSHA256"
        private const val PBKDF2_ITERATIONS = 100_000  // OWASP 2023
        private const val KEY_LENGTH_BITS = 256
        private const val SALT_SIZE = 16  // 128 bits
    }
}
```

**Características de Seguridad**:
- **100,000 iteraciones**: Recomendación OWASP 2023 para hacer fuerza bruta inviable
- **Salt de 16 bytes**: Generado con `SecureRandom` para cada usuario
- **Limpieza de memoria**: `CharArray` se limpia después de usar para evitar fugas
- **SHA-256**: Función hash criptográfica segura

---

### CipherManager

**Archivo**: `security/CipherManager.kt`

Cifra/descifra la clave derivada usando RSA del Keystore.

```kotlin
class CipherManager {
    companion object {
        private const val RSA_TRANSFORMATION = "RSA/ECB/PKCS1Padding"
    }
    
    fun encrypt(keyToEncrypt: ByteArray, publicKey: PublicKey): ByteArray {
        val cipher = Cipher.getInstance(RSA_TRANSFORMATION)
        cipher.init(Cipher.ENCRYPT_MODE, publicKey)
        return cipher.doFinal(keyToEncrypt)
    }
    
    fun decrypt(encryptedData: ByteArray, privateKey: PrivateKey): ByteArray {
        val cipher = Cipher.getInstance(RSA_TRANSFORMATION)
        cipher.init(Cipher.DECRYPT_MODE, privateKey)
        return cipher.doFinal(encryptedData)
    }
}
```

**Parámetros RSA**:
| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| Algoritmo | RSA | Criptografía asimétrica |
| Padding | PKCS1Padding | Compatible universalmente en Android |
| Clave | 2048 bits | Estándar de seguridad |
| Uso | Cifrar clave derivada | Protege la clave maestra |

**Logging Detallado** (para debugging):
```kotlin
fun decrypt(encryptedData: ByteArray, privateKey: PrivateKey): ByteArray {
    Log.d(TAG, "========== INICIO DESCIFRADO RSA ==========")
    Log.d(TAG, "Datos cifrados: ${encryptedData.size} bytes")
    Log.d(TAG, "Algoritmo: $RSA_TRANSFORMATION")
    Log.d(TAG, "Clave privada algoritmo: ${privateKey.algorithm}")
    Log.d(TAG, "Clave privada formato: ${privateKey.format}")
    
    val cipher = Cipher.getInstance(RSA_TRANSFORMATION)
    cipher.init(Cipher.DECRYPT_MODE, privateKey)
    
    val result = cipher.doFinal(encryptedData)
    Log.d(TAG, "Datos descifrados correctamente (${result.size} bytes)")
    Log.d(TAG, "========== FIN DESCIFRADO RSA ==========")
    return result
}
```

---

### DataCipher

**Archivo**: `security/DataCipher.kt`

Cifra/descifra campos individuales (password, notas) con AES-256-GCM.

```kotlin
class DataCipher {
    private val secureRandom = SecureRandom()
    private var lastIv: ByteArray? = null
    
    fun encrypt(plaintext: String, key: ByteArray): ByteArray {
        require(key.size == KEY_SIZE_BYTES)
        
        // Generar IV aleatorio de 12 bytes (96 bits)
        val iv = ByteArray(IV_SIZE)
        secureRandom.nextBytes(iv)
        lastIv = iv
        
        val cipher = Cipher.getInstance(AES_GCM_TRANSFORMATION)
        val keySpec = SecretKeySpec(key, AES_ALGORITHM)
        val gcmSpec = GCMParameterSpec(TAG_LENGTH_BITS, iv)
        
        cipher.init(Cipher.ENCRYPT_MODE, keySpec, gcmSpec)
        val ciphertext = cipher.doFinal(plaintext.toByteArray(Charsets.UTF_8))
        
        // Prependir IV al ciphertext
        return iv + ciphertext
    }
    
    fun decrypt(ciphertext: ByteArray, key: ByteArray): String {
        require(key.size == KEY_SIZE_BYTES)
        require(ciphertext.size > IV_SIZE)
        
        // Extraer IV (primeros 12 bytes)
        val iv = ciphertext.copyOfRange(0, IV_SIZE)
        val actualCiphertext = ciphertext.copyOfRange(IV_SIZE, ciphertext.size)
        
        val cipher = Cipher.getInstance(AES_GCM_TRANSFORMATION)
        val keySpec = SecretKeySpec(key, AES_ALGORITHM)
        val gcmSpec = GCMParameterSpec(TAG_LENGTH_BITS, iv)
        
        cipher.init(Cipher.DECRYPT_MODE, keySpec, gcmSpec)
        val plaintextBytes = cipher.doFinal(actualCiphertext)
        return String(plaintextBytes, Charsets.UTF_8)
    }
    
    companion object {
        private const val AES_ALGORITHM = "AES"
        private const val AES_GCM_TRANSFORMATION = "AES/GCM/NoPadding"
        private const val KEY_SIZE_BYTES = 32  // 256 bits
        private const val IV_SIZE = 12         // 96 bits (recomendado para GCM)
        private const val TAG_LENGTH_BITS = 128 // Tag de autenticación
    }
}
```

**Características de AES-GCM**:
- **Galois/Counter Mode**: Proporciona confidencialidad + autenticidad
- **Tag de 128 bits**: Detecta cualquier modificación de datos
- **IV único por operación**: Previene ataques de replay
- **No padding**: Modo de flujo, más eficiente

---

### SecureStorage

**Archivo**: `security/SecureStorage.kt`

Almacena metadata de cifrado en SharedPreferences.

```kotlin
class SecureStorage(context: Context) {
    private val prefs: SharedPreferences = context.getSharedPreferences(
        PREFS_NAME, Context.MODE_PRIVATE
    )
    
    fun saveSalt(salt: ByteArray) {
        val encoded = Base64.encodeToString(salt, Base64.NO_WRAP)
        prefs.edit().putString(KEY_SALT, encoded).apply()
    }
    
    fun saveEncryptedKey(encryptedKey: ByteArray) {
        val encoded = Base64.encodeToString(encryptedKey, Base64.NO_WRAP)
        prefs.edit().putString(KEY_ENCRYPTED_KEY, encoded).apply()
    }
    
    fun saveIV(iv: ByteArray) {
        val encoded = Base64.encodeToString(iv, Base64.NO_WRAP)
        prefs.edit().putString(KEY_IV, encoded).apply()
    }
    
    fun getSalt(): ByteArray? {
        val encoded = prefs.getString(KEY_SALT, null) ?: return null
        return Base64.decode(encoded, Base64.NO_WRAP)
    }
    
    fun getEncryptedKey(): ByteArray? {
        val encoded = prefs.getString(KEY_ENCRYPTED_KEY, null) ?: return null
        return Base64.decode(encoded, Base64.NO_WRAP)
    }
    
    fun hasStoredData(): Boolean {
        return getSalt() != null && getEncryptedKey() != null
    }
    
    companion object {
        const val PREFS_NAME = "secure_storage"
        private const val KEY_SALT = "salt"
        private const val KEY_ENCRYPTED_KEY = "encrypted_key"
        private const val KEY_IV = "iv"
    }
}
```

**Datos Almacenados**:
| Clave | Contenido | Propósito |
|-------|-----------|-----------|
| `salt` | 16 bytes (Base64) | Derivar clave del password |
| `encrypted_key` | Clave cifrada RSA | Clave maestra protegida |
| `iv` | 12 bytes (Base64) | Último IV usado (referencia) |

---

### BiometricAuthenticator

**Archivo**: `security/BiometricAuthenticator.kt`

Wrapper de BiometricPrompt para autenticación biométrica.

```kotlin
sealed class BiometricStatus {
    object Available : BiometricStatus()      // Hardware + configurado
    object NotAvailable : BiometricStatus()   // Sin hardware
    object NotEnrolled : BiometricStatus()    // Sin biometría registrada
}

class BiometricAuthenticator {
    fun canAuthenticate(context: Context): BiometricStatus {
        val biometricManager = BiometricManager.from(context)
        
        return when (biometricManager.canAuthenticate(
            BiometricManager.Authenticators.BIOMETRIC_STRONG
        )) {
            BiometricManager.BIOMETRIC_SUCCESS -> BiometricStatus.Available
            BiometricManager.BIOMETRIC_ERROR_NO_HARDWARE -> BiometricStatus.NotAvailable
            BiometricManager.BIOMETRIC_ERROR_NONE_ENROLLED -> BiometricStatus.NotEnrolled
            else -> BiometricStatus.NotAvailable
        }
    }
    
    fun authenticate(
        fragment: Fragment,
        onAuthSuccess: () -> Unit,
        onAuthError: (String) -> Unit,
        onCancellation: () -> Unit = {}
    ) {
        val executor = ContextCompat.getMainExecutor(fragment.requireContext())
        
        val biometricPrompt = BiometricPrompt(
            fragment, executor,
            object : BiometricPrompt.AuthenticationCallback() {
                override fun onAuthenticationSucceeded(result: AuthenticationResult) {
                    onAuthSuccess()
                }
                
                override fun onAuthenticationFailed() {
                    onAuthError("Biometría no reconocida. Inténtalo de nuevo.")
                }
                
                override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
                    if (errorCode == BiometricPrompt.ERROR_USER_CANCELED) {
                        onCancellation()
                    } else {
                        onAuthError(errString.toString())
                    }
                }
            }
        )
        
        val promptInfo = BiometricPrompt.PromptInfo.Builder()
            .setTitle("Autenticación requerida")
            .setSubtitle("Usa tu huella o rostro para desbloquear PasswordManager")
            .setNegativeButtonText("Usar contraseña")
            .build()
        
        biometricPrompt.authenticate(promptInfo)
    }
}
```

**Tipos Biométricos Soportados**:
- Huella dactilar (Fingerprint)
- Reconocimiento facial (Face Unlock)
- Iris scanner (dispositivos compatibles)

**Estados**:
| Estado | Descripción | Acción |
|--------|-------------|--------|
| `Available` | Hardware presente + biometría registrada | Permitir autenticación |
| `NotAvailable` | Sin hardware biométrico | Usar contraseña maestra |
| `NotEnrolled` | Hardware existe, sin biometría | Guijar a configuración |

---

## Flujo de Cifrado Completo

### Configuración Inicial (Setup)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Usuario crea password maestro: "MiPassword123!"              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. PasswordDeriver.generateSalt()                               │
│    Salt: [0x3A, 0x7F, 0x1B, ...] (16 bytes aleatorios)          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. PasswordDeriver.deriveKey(password, salt)                    │
│    PBKDF2-HMAC-SHA256, 100,000 iteraciones                      │
│    Clave derivada: [0x8B, 0x2C, 0x9D, ...] (32 bytes)           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. KeystoreManager.generateKeyIfNeeded()                        │
│    Genera par RSA-2048 en Android Keystore                      │
│    Clave pública:  ---BEGIN PUBLIC KEY---...                    │
│    Clave privada:  (requiere biometría)                         │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. CipherManager.encrypt(derivedKey, publicKey)                 │
│    RSA/ECB/PKCS1Padding                                         │
│    Clave cifrada: [0x4D, 0x8E, 0x2A, ...]                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. SecureStorage.saveSalt(salt)                                 │
│    SecureStorage.saveEncryptedKey(encryptedKey)                 │
│    SharedPreferences:                                           │
│      salt = "On9/GhsfGxY=..."                                   │
│      encrypted_key = "TYuI8HbsdfG..."                           │
└─────────────────────────────────────────────────────────────────┘
```

### Autenticación (Login)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Usuario ingresa password: "MiPassword123!"                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. Leer salt de SecureStorage                                   │
│    salt = [0x3A, 0x7F, 0x1B, ...]                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. PasswordDeriver.deriveKey(password, salt)                    │
│    Clave candidata: [0x8B, 0x2C, 0x9D, ...]                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. Leer encryptedKey de SecureStorage                           │
│    encryptedKey = [0x4D, 0x8E, 0x2A, ...]                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. BiometricAuthenticator.authenticate()                        │
│    Muestra BiometricPrompt                                      │
│    Usuario autentica con huella                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. KeystoreManager.getPrivateKey()                              │
│    Desbloquea clave privada (requiere biometría)                │
│    privateKey = AndroidKeyStore private key                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. CipherManager.decrypt(encryptedKey, privateKey)              │
│    RSA descifrado                                               │
│    Clave almacenada: [0x8B, 0x2C, 0x9D, ...]                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 8. Comparar claves                                              │
│    candidateKey.contentEquals(storedKey)                        │
│    ✓ Iguales → Autenticación exitosa                           │
│    ✗ Diferentes → SecurityException                             │
└─────────────────────────────────────────────────────────────────┘
```

### Cifrado de Contraseña (Guardar)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Usuario guarda password: "SuperSecret123!"                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. Obtener masterKey de sesión                                  │
│    masterKey = [0x8B, 0x2C, 0x9D, ...]                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. DataCipher.encrypt(password, masterKey)                      │
│    a) Generar IV: [0x5A, 0x3B, 0x7C, ...] (12 bytes)            │
│    b) AES-256-GCM cifrado                                       │
│    c) ciphertext = [0x9E, 0x4F, 0x1D, ...]                      │
│    d) Retornar: IV + ciphertext                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. Codificar Base64                                             │
│    Base64.encode(IV + ciphertext)                               │
│    "WjtXfJ4K9mLpQ2vN8xR3..."                                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. PasswordRepositoryImpl.insertEntry()                         │
│    INSERT INTO password_entries (password, ...)                 │
│    VALUES ('WjtXfJ4K9mLpQ2vN8xR3...', ...)                      │
└─────────────────────────────────────────────────────────────────┘
```

### Descifrado de Contraseña (Leer)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. PasswordListScreen observa getAllEntries()                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. PasswordEntryDao.getAll()                                    │
│    SELECT * FROM password_entries                               │
│    Retorna: PasswordEntryEntity(password="WjtXfJ4K9m...")       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. PasswordRepositoryImpl.decryptEntry()                        │
│    a) Decodificar Base64: Base64.decode("WjtXfJ4K9m...")        │
│    b) Extraer IV: primeros 12 bytes                             │
│    c) Extraer ciphertext: resto                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. DataCipher.decrypt(ciphertext, masterKey)                    │
│    a) AES-256-GCM descifrado                                    │
│    b) Verificar tag de autenticación                            │
│    c) Retornar plaintext: "SuperSecret123!"                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. Construir PasswordEntry (dominio)                            │
│    PasswordEntry(password="SuperSecret123!", ...)               │
│    Emite a Flow → ViewModel → UI                                │
└─────────────────────────────────────────────────────────────────┘
```

## Testing de Seguridad

### Unit Tests para PasswordDeriver

```kotlin
class PasswordDeriverTest {
    
    private val passwordDeriver = PasswordDeriver()
    
    @Test
    fun `salt should be 16 bytes`() {
        // When
        val salt = passwordDeriver.generateSalt()
        
        // Then
        assertEquals(16, salt.size)
    }
    
    @Test
    fun `derived key should be 32 bytes (256 bits)`() {
        // Given
        val password = "TestPassword123!".toCharArray()
        val salt = passwordDeriver.generateSalt()
        
        // When
        val key = passwordDeriver.deriveKey(password, salt)
        
        // Then
        assertEquals(32, key.size)
    }
    
    @Test
    fun `same password and salt should produce same key`() {
        // Given
        val password = "TestPassword123!".toCharArray()
        val salt = passwordDeriver.generateSalt()
        
        // When
        val key1 = passwordDeriver.deriveKey(password, salt)
        val key2 = passwordDeriver.deriveKey(password, salt)
        
        // Then
        assertTrue(key1.contentEquals(key2))
    }
    
    @Test
    fun `different salt should produce different key`() {
        // Given
        val password = "TestPassword123!".toCharArray()
        val salt1 = passwordDeriver.generateSalt()
        val salt2 = passwordDeriver.generateSalt()
        
        // When
        val key1 = passwordDeriver.deriveKey(password, salt1)
        val key2 = passwordDeriver.deriveKey(password, salt2)
        
        // Then
        assertFalse(key1.contentEquals(key2))
    }
}
```

### Unit Tests para DataCipher

```kotlin
class DataCipherTest {
    
    private val dataCipher = DataCipher()
    private val testKey = ByteArray(32) { 0x42 }  // Clave de prueba
    
    @Test
    fun `encrypt and decrypt should return original text`() {
        // Given
        val plaintext = "SecretPassword123!"
        
        // When
        val encrypted = dataCipher.encrypt(plaintext, testKey)
        val decrypted = dataCipher.decrypt(encrypted, testKey)
        
        // Then
        assertEquals(plaintext, decrypted)
    }
    
    @Test
    fun `same plaintext should produce different ciphertext`() {
        // Given
        val plaintext = "SecretPassword123!"
        
        // When
        val encrypted1 = dataCipher.encrypt(plaintext, testKey)
        val encrypted2 = dataCipher.encrypt(plaintext, testKey)
        
        // Then
        // IV aleatorio hace que cada cifrado sea único
        assertFalse(encrypted1.contentEquals(encrypted2))
    }
    
    @Test
    fun `different key should fail decryption`() {
        // Given
        val plaintext = "SecretPassword123!"
        val wrongKey = ByteArray(32) { 0x00 }
        val encrypted = dataCipher.encrypt(plaintext, testKey)
        
        // When / Then
        assertThrows(BadPaddingException::class.java) {
            dataCipher.decrypt(encrypted, wrongKey)
        }
    }
}
```

## Consideraciones de Seguridad

### 1. Gestión de Memoria

```kotlin
// ✅ CORRECTO: Usar CharArray y limpiar
fun deriveKey(password: CharArray, salt: ByteArray): ByteArray {
    try {
        val keySpec = PBEKeySpec(password, salt, iterations, keyLength)
        // ... usar keySpec
        return keyBytes
    } finally {
        keySpec.clearPassword()  // Limpiar memoria
        password.fillPassword()   // Limpiar password
    }
}

// ❌ INCORRECTO: Usar String (inmutable en memoria)
fun deriveKey(password: String, salt: ByteArray): ByteArray {
    // El password permanece en memoria hasta GC
}
```

### 2. Almacenamiento Seguro

| Dato | Ubicación | Protección |
|------|-----------|------------|
| Salt | SharedPreferences | No sensible, puede ser público |
| Clave cifrada | SharedPreferences | Cifrada con RSA (Keystore) |
| Clave privada | Android Keystore | Hardware-backed, requiere biometría |
| Contraseñas | Room Database | Cifradas con AES-256-GCM |
| Notas | Room Database | Cifradas con AES-256-GCM |

### 3. Protección contra Ataques

| Ataque | Mitigación |
|--------|------------|
| Fuerza bruta | PBKDF2 100,000 iteraciones |
| Rainbow tables | Salt único por usuario |
| Extracción de datos | AES-256-GCM con tag de autenticación |
| Extracción de claves | Android Keystore hardware-backed |
| Replay attacks | IV único por operación |
| Modificación de datos | GCM tag detecta alteraciones |

## Referencias

- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Android Keystore System](https://developer.android.com/security/keystore)
- [AES-GCM Implementation Guide](https://developer.android.com/security/using-cryptography)
- [BiometricPrompt Documentation](https://developer.android.com/reference/androidx/biometric/BiometricPrompt)

---

**Documentación Relacionada:**
- [Arquitectura](../arquitectura/overview.md)
- [Dominio](../domain/overview.md)
- [Presentation](../presentation/overview.md)

