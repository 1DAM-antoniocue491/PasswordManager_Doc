# Componentes de Seguridad

La capa de seguridad maneja cifrado, derivación de claves y autenticación biométrica.

---

## PasswordDeriver

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

## CipherManager

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

---

## DataCipher

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

## SecureStorage

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

## BiometricAuthenticator

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

**Errores Biométricos**:

| Código | Descripción |
|--------|-------------|
| `BIOMETRIC_ERROR_NO_HARDWARE` | Sin sensor biométrico |
| `BIOMETRIC_ERROR_NONE_ENROLLED` | Sin biometría configurada |
| `ERROR_USER_CANCELED` | Usuario canceló la autenticación |

---

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
        assertTrue(key1 contentEquals key2)
    }
    
    @Test
    fun `different salt should produce different keys`() {
        // Given
        val password = "TestPassword123!".toCharArray()
        val salt1 = passwordDeriver.generateSalt()
        val salt2 = passwordDeriver.generateSalt()
        
        // When
        val key1 = passwordDeriver.deriveKey(password, salt1)
        val key2 = passwordDeriver.deriveKey(password, salt2)
        
        // Then
        assertFalse(key1 contentEquals key2)
    }
}
```
