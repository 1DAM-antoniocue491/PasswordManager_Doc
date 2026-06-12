# Data Access Objects (DAO)

Los DAOs definen las operaciones de acceso a datos para Room.

---

## PasswordEntryDao

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

---

## CategoryDao

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

---

## SettingsDao

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

---

## Testing de DAOs

### Unit Tests para PasswordEntryDao

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
