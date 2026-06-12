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

## Referencias

- [Room Documentation](https://developer.android.com/training/data-storage/room)
- [Flow Documentation](https://kotlinlang.org/docs/flow.html)
- [DataStore Documentation](https://developer.android.com/topic/libraries/architecture/datastore)

---

**Documentación Relacionada:**
- [Arquitectura](../arquitectura/overview.md)
- [Seguridad](../security/overview.md)
- [Dominio](../domain/overview.md)
