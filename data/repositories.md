# Repositorios (Implementaciones)

Las implementaciones de repositorios conectan los DAOs con la capa de dominio.

---

## PasswordRepositoryImpl

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
            username = entity.username,
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

---

## AuthRepositoryImpl

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

---

## CategoryRepositoryImpl

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

---

## SettingsRepositoryImpl

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

---

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
