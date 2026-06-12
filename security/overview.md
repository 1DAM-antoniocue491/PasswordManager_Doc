# Sistema de Seguridad

## Visión General

Password Manager implementa un sistema de seguridad de múltiples capas para proteger las contraseñas del usuario. El sistema combina:

1. **Cifrado Híbrido**: RSA + AES-GCM
2. **Derivación de Claves**: PBKDF2-HMAC-SHA256
3. **Almacenamiento Seguro**: Android Keystore
4. **Autenticación Biométrica**: Huella dactilar/rostro

## Arquitectura de Seguridad

```
┌─────────────────────────────────────────────────────────────────┐
│                    Usuario (Password Maestro)                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   PasswordDeriver (PBKDF2)                       │
│              100,000 iteraciones, salt de 16 bytes               │
│                    ↓ Clave derivada (256 bits)                   │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│   CipherManager (RSA)   │     │    DataCipher (AES)     │
│  Cifra la clave con     │     │  Cifra los datos con    │
│  clave pública Keystore │     │  clave derivada (GCM)   │
└─────────────────────────┘     └─────────────────────────┘
              │                               │
              ▼                               ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│   SecureStorage         │     │   PasswordEntryEntity   │
│  Guarda clave cifrada   │     │  Guarda password cifrada│
│  + salt + IV            │     │  + notes cifradas       │
└─────────────────────────┘     └─────────────────────────┘
```

## Componentes de Seguridad

### 1. PasswordDeriver

**Propósito**: Derivar una clave simétrica de 256 bits a partir del password maestro del usuario.

**Algoritmo**: PBKDF2-HMAC-SHA256 con 100,000 iteraciones (recomendación OWASP 2023).

```kotlin
class PasswordDeriver {
    companion object {
        private const val PBKDF2_ALGORITHM = "PBKDF2withHmacSHA256"
        private const val PBKDF2_ITERATIONS = 100_000  // OWASP 2023
        private const val KEY_LENGTH_BITS = 256
        private const val SALT_SIZE = 16  // 128 bits
    }
    
    fun deriveKey(password: CharArray, salt: ByteArray): ByteArray {
        val keySpec = PBEKeySpec(password, salt, PBKDF2_ITERATIONS, KEY_LENGTH_BITS)
        val secretKeyFactory = SecretKeyFactory.getInstance(PBKDF2_ALGORITHM)
        return secretKeyFactory.generateSecret(keySpec).encoded
    }
}
```

**Características**:
- **Salt aleatorio**: 16 bytes generados con `SecureRandom`
- **Password en CharArray**: Se limpia después de usar para evitar fugas en memoria
- **Iteraciones**: 100,000 para hacer fuerza bruta computacionalmente inviable

### 2. KeystoreManager

**Propósito**: Gestionar claves RSA en el Android Keystore hardware-backed.

**Características**:
- Genera par de claves RSA-2048
- Clave privada requiere autenticación biométrica
- Almacenamiento en hardware (TEE/SE) cuando está disponible

```kotlin
class KeystoreManager {
    companion object {
        private const val KEYSTORE_PROVIDER = "AndroidKeyStore"
        private const val KEY_ALIAS = "password_manager_key"
    }
    
    fun generateKeyIfNeeded() {
        if (!keyStore.containsAlias(KEY_ALIAS)) {
            val keyPairGenerator = KeyPairGenerator.getInstance(
                KeyProperties.KEY_ALGORITHM_RSA, KEYSTORE_PROVIDER
            )
            keyPairGenerator.initialize(
                KeyGenParameterSpec.Builder(
                    KEY_ALIAS,
                    KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
                )
                    .setBlockModes(KeyProperties.BLOCK_MODE_ECB)
                    .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_RSA_PKCS1)
                    .setUserAuthenticationRequired(true)  // Requiere biometría
                    .setKeySize(2048)
                    .build()
            )
            keyPairGenerator.generateKeyPair()
        }
    }
}
```

**Parámetros de la clave**:
| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| Algoritmo | RSA | Criptografía asimétrica |
| Tamaño | 2048 bits | Nivel de seguridad estándar |
| Padding | PKCS1 | Compatible con todos los dispositivos |
| User Authentication | true | Requiere biometría para usar clave privada |

### 3. CipherManager

**Propósito**: Cifrar/descifrar la clave derivada usando RSA.

**Algoritmo**: RSA/ECB/PKCS1Padding

```kotlin
class CipherManager {
    fun encrypt(keyToEncrypt: ByteArray, publicKey: PublicKey): ByteArray {
        val cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding")
        cipher.init(Cipher.ENCRYPT_MODE, publicKey)
        return cipher.doFinal(keyToEncrypt)
    }
    
    fun decrypt(encryptedData: ByteArray, privateKey: PrivateKey): ByteArray {
        val cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding")
        cipher.init(Cipher.DECRYPT_MODE, privateKey)
        return cipher.doFinal(encryptedData)
    }
}
```

**Flujo**:
1. Cifrado: Clave derivada → RSA (clave pública) → Clave cifrada
2. Descifrado: Clave cifrada → RSA (clave privada + biometría) → Clave derivada

### 4. DataCipher

**Propósito**: Cifrar/descifrar campos sensibles (password, notas) con AES-GCM.

**Algoritmo**: AES-256-GCM (Galois/Counter Mode)

```kotlin
class DataCipher {
    companion object {
        private const val AES_GCM_TRANSFORMATION = "AES/GCM/NoPadding"
        private const val KEY_SIZE_BYTES = 32  // 256 bits
        private const val IV_SIZE = 12         // 96 bits (recomendado para GCM)
        private const val TAG_LENGTH_BITS = 128 // Tag de autenticación
    }
    
    fun encrypt(plaintext: String, key: ByteArray): ByteArray {
        val iv = ByteArray(IV_SIZE)
        secureRandom.nextBytes(iv)  // IV aleatorio por operación
        
        val cipher = Cipher.getInstance(AES_GCM_TRANSFORMATION)
        val keySpec = SecretKeySpec(key, "AES")
        val gcmSpec = GCMParameterSpec(TAG_LENGTH_BITS, iv)
        
        cipher.init(Cipher.ENCRYPT_MODE, keySpec, gcmSpec)
        val ciphertext = cipher.doFinal(plaintext.toByteArray(Charsets.UTF_8))
        
        return iv + ciphertext  // Prependir IV para almacenar
    }
    
    fun decrypt(ciphertext: ByteArray, key: ByteArray): String {
        val iv = ciphertext.copyOfRange(0, IV_SIZE)
        val actualCiphertext = ciphertext.copyOfRange(IV_SIZE, ciphertext.size)
        
        val cipher = Cipher.getInstance(AES_GCM_TRANSFORMATION)
        val keySpec = SecretKeySpec(key, "AES")
        val gcmSpec = GCMParameterSpec(TAG_LENGTH_BITS, iv)
        
        cipher.init(Cipher.DECRYPT_MODE, keySpec, gcmSpec)
        val plaintextBytes = cipher.doFinal(actualCiphertext)
        return String(plaintextBytes, Charsets.UTF_8)
    }
}
```

**Características de AES-GCM**:
- **Confidencialidad**: Cifrado AES-256
- **Autenticidad**: Tag de 128 bits detecta modificación de datos
- **IV único**: 12 bytes aleatorios por cada operación de cifrado
- **No padding**: GCM es modo de flujo, no requiere padding

### 5. SecureStorage

**Propósito**: Almacenar metadata de cifrado en SharedPreferences.

**Datos almacenados**:
| Clave | Tipo | Descripción |
|-------|------|-------------|
| `salt` | Base64 | Salt usado para derivar clave |
| `encrypted_key` | Base64 | Clave derivada cifrada con RSA |
| `iv` | Base64 | Último IV usado (para referencia) |

```kotlin
class SecureStorage(context: Context) {
    private val prefs: SharedPreferences = context.getSharedPreferences(
        "secure_storage", Context.MODE_PRIVATE
    )
    
    fun saveSalt(salt: ByteArray) {
        prefs.edit().putString("salt", Base64.encodeToString(salt, Base64.NO_WRAP)).apply()
    }
    
    fun saveEncryptedKey(encryptedKey: ByteArray) {
        prefs.edit().putString("encrypted_key", 
            Base64.encodeToString(encryptedKey, Base64.NO_WRAP)).apply()
    }
    
    fun getSalt(): ByteArray? {
        val encoded = prefs.getString("salt", null) ?: return null
        return Base64.decode(encoded, Base64.NO_WRAP)
    }
    
    fun getEncryptedKey(): ByteArray? {
        val encoded = prefs.getString("encrypted_key", null) ?: return null
        return Base64.decode(encoded, Base64.NO_WRAP)
    }
}
```

### 6. BiometricAuthenticator

**Propósito**: Wrapper de BiometricPrompt para autenticación biométrica.

**Tipos de autenticación soportados**:
- Huella dactilar (Fingerprint)
- Reconocimiento facial (Face Unlock)
- Iris scanner (en dispositivos compatibles)

```kotlin
class BiometricAuthenticator {
    fun canAuthenticate(context: Context): BiometricStatus {
        val biometricManager = BiometricManager.from(context)
        return when (biometricManager.canAuthenticate(BiometricManager.Authenticators.BIOMETRIC_STRONG)) {
            BiometricManager.BIOMETRIC_SUCCESS -> BiometricStatus.Available
            BiometricManager.BIOMETRIC_ERROR_NO_HARDWARE -> BiometricStatus.NotAvailable
            BiometricManager.BIOMETRIC_ERROR_NONE_ENROLLED -> BiometricStatus.NotEnrolled
            else -> BiometricStatus.NotAvailable
        }
    }
    
    fun authenticate(
        fragment: Fragment,
        onAuthSuccess: () -> Unit,
        onAuthError: (String) -> Unit
    ) {
        val biometricPrompt = BiometricPrompt(fragment, executor, callback)
        val promptInfo = BiometricPrompt.PromptInfo.Builder()
            .setTitle("Autenticación requerida")
            .setSubtitle("Usa tu huella o rostro para desbloquear")
            .setNegativeButtonText("Usar contraseña")
            .build()
        biometricPrompt.authenticate(promptInfo)
    }
}
```

## Flujo de Configuración Inicial

```
1. Usuario crea password maestro
         ↓
2. PasswordDeriver.generateSalt() → Salt aleatorio de 16 bytes
         ↓
3. PasswordDeriver.deriveKey(password, salt) → Clave de 256 bits
         ↓
4. KeystoreManager.generateKeyIfNeeded() → Claves RSA en Keystore
         ↓
5. CipherManager.encrypt(derivedKey, publicKey) → Clave cifrada
         ↓
6. SecureStorage.saveSalt(salt)
   SecureStorage.saveEncryptedKey(encryptedKey)
         ↓
7. ¡Configuración completada!
```

## Flujo de Autenticación

```
1. Usuario ingresa password maestro
         ↓
2. Leer salt de SecureStorage
         ↓
3. PasswordDeriver.deriveKey(password, salt) → Clave candidata
         ↓
4. Leer encryptedKey de SecureStorage
         ↓
5. Solicitar autenticación biométrica
         ↓
6. CipherManager.decrypt(encryptedKey, privateKey) → Clave original
         ↓
7. Comparar clave candidata con clave original
         ↓
8. Si coinciden → Autenticación exitosa
         ↓
9. Mantener clave en memoria para cifrar/descifrar datos
```

## Flujo de Cifrado de Datos

```
Para guardar una contraseña:

1. Obtener clave maestra de memoria
         ↓
2. DataCipher.encrypt(password, masterKey)
   - Generar IV aleatorio de 12 bytes
   - Cifrar con AES-256-GCM
   - Retornar: IV + ciphertext + tag
         ↓
3. Base64.encode(resultado)
         ↓
4. Guardar en Room Database
```

## Flujo de Descifrado de Datos

```
Para leer una contraseña:

1. Leer datos cifrados de Room (Base64)
         ↓
2. Base64.decode() → IV + ciphertext + tag
         ↓
3. Obtener clave maestra de memoria
         ↓
4. DataCipher.decrypt(datos, masterKey)
   - Extraer IV (primeros 12 bytes)
   - Verificar tag de autenticación
   - Descifrar ciphertext
         ↓
5. Retornar password en texto plano
```

## Consideraciones de Seguridad

### 1. Manejo de Memoria

- **CharArray para passwords**: Se limpia explícitamente después de usar
- **Clave en memoria**: Solo mientras la sesión está activa
- **Cero en cleanup**: Al cerrar sesión, la clave se elimina de memoria

```kotlin
// Limpieza segura de CharArray
private fun CharArray.fillPassword() {
    java.util.Arrays.fill(this, ' ')
}
```

### 2. Protección contra Fuerza Bruta

| Medida | Implementación |
|--------|----------------|
| Iteraciones PBKDF2 | 100,000 (OWASP 2023) |
| Salt | 16 bytes aleatorios por usuario |
| Complejidad password | Validada en UI (mínimo 8 caracteres) |
| Timeout de intentos | AutoLockManager bloquea después de inactividad |

### 3. Protección de Datos en Reposo

| Dato | Cifrado | Almacenamiento |
|------|---------|----------------|
| Password maestro | No se almacena | Solo en memoria RAM |
| Clave derivada | RSA-2048 | SecureStorage |
| Contraseñas | AES-256-GCM | Room Database |
| Notas | AES-256-GCM | Room Database |
| Salt | Sin cifrar | SecureStorage |

### 4. Autenticación Biométrica

**Nivel de seguridad**: `BIOMETRIC_STRONG`

Requiere:
- Hardware biométrico Clase 2 o superior
- Datos biométricos almacenados en TEE/SE
- Liveness detection (detección de vida)

**Fallback**: Contraseña maestra si biometría no está disponible

### 5. AutoLock Manager

Gestiona el bloqueo automático por inactividad:

```kotlin
class AutoLockManager {
    private var lastAccessTime: Long = 0
    private var lockTimeoutMs: Long = 300_000  // 5 minutos por defecto
    
    fun initialize() {
        // Registrar callback de ciclo de vida
        ProcessLifecycleOwner.get().lifecycle.addObserver(this)
    }
    
    override fun onStop(source: LifecycleOwner) {
        // La app pasó a segundo plano, iniciar timer
        lastAccessTime = System.currentTimeMillis()
    }
    
    override fun onStart(source: LifecycleOwner) {
        // La app volvió a primer plano
        if (System.currentTimeMillis() - lastAccessTime > lockTimeoutMs) {
            // Requiere autenticación de nuevo
            requireAuthentication()
        }
    }
}
```

## Algoritmos y Parámetros

| Componente | Algoritmo | Parámetros |
|------------|-----------|------------|
| Derivación de clave | PBKDF2-HMAC-SHA256 | 100,000 iteraciones, 256 bits |
| Cifrado asimétrico | RSA-PKCS1 | 2048 bits |
| Cifrado simétrico | AES-GCM | 256 bits, IV 96 bits, tag 128 bits |
| Salt | SecureRandom | 128 bits (16 bytes) |
| Hash de autenticación | HMAC-SHA256 | 256 bits |

## Vulnerabilidades Mitigadas

| Vulnerabilidad | Mitigación |
|----------------|------------|
| Fuerza bruta offline | PBKDF2 con 100k iteraciones |
| Rainbow tables | Salt único por usuario |
| Ataques de canal lateral | Claves en Keystore hardware-backed |
| Modificación de datos | AES-GCM con tag de autenticación |
| Replay attacks | IV único por operación de cifrado |
| Memory dumping | Limpieza explícita de CharArrays |
| Root/Jailbreak | Keystore requiere hardware seguro |

## Recomendaciones para el Usuario

1. **Password maestro fuerte**: Mínimo 12 caracteres, combinación de mayúsculas, minúsculas, números y símbolos
2. **Biometría habilitada**: Para acceso rápido y seguro
3. **Backup cifrado**: Exportar backup periódicamente
4. **No rootear el dispositivo**: Compromete la seguridad del Keystore

## Referencias

- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Android Keystore System](https://developer.android.com/training/articles/keystore)
- [NIST SP 800-132](https://csrc.nist.gov/publications/detail/sp/800-132/final) - PBKDF Recommendation
- [AES-GCM Specification](https://csrc.nist.gov/publications/detail/sp/800-38d/final)

---

**Documentación Relacionada:**
- [Arquitectura](../arquitectura/overview.md)
- [Capa de Datos](../data/overview.md)
- [Repository Pattern](../domain/overview.md)
