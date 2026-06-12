# Inicio

## Visión General

Password Manager es una aplicación Android nativa para la gestión segura de contraseñas, desarrollada con **Jetpack Compose** y siguiendo arquitecturas modernas de desarrollo Android.

### Características Principales

* **Almacenamiento Seguro**: Las contraseñas se cifran utilizando AES-GCM a través de Android Keystore
* **Autenticación Biométrica**: Soporte para huella dactilar y reconocimiento facial
* **Gestión de Categorías**: Organización de contraseñas por categorías personalizadas
* **Generador de Contraseñas**: Creación de contraseñas seguras con opciones configurables
* **Búsqueda y Filtrado**: Búsqueda en tiempo real y filtrado por categorías
* **Favoritos**: Marcado de contraseñas frecuentes para acceso rápido
* **Copia de Seguridad**: Exportación e importación de datos cifrados
* **Auditoría de Seguridad**: Detección de contraseñas débiles
* **Estadísticas**: Resumen del estado de seguridad de las contraseñas

## Stack Tecnológico

| Componente                    | Tecnología                 |
| ----------------------------- | -------------------------- |
| **UI Framework**              | Jetpack Compose            |
| **Arquitectura**              | Clean Architecture + MVVM  |
| **Base de Datos**             | Room Database              |
| **Inyección de Dependencias** | Koin                       |
| **Cifrado**                   | Android Keystore + AES-GCM |
| **Navegación**                | Navigation Compose         |
| **Preferencias**              | DataStore                  |
| **Serialización**             | Kotlinx Serialization      |

## Estructura del Proyecto

```
app/
├── data/                          # Capa de Datos
│   ├── local/                     # Persistencia local (Room)
│   │   ├── dao/                   # Data Access Objects
│   │   ├── entity/                # Entidades de base de datos
│   │   ├── converter/             # Conversores de tipo
│   │   └── PasswordDatabase.kt    # Configuración de Room
│   ├── repository/                # Implementaciones de repositorios
│   └── security/                  # Componentes de seguridad
│       ├── KeystoreManager.kt     # Gestión de claves Android
│       ├── CipherManager.kt       # Cifrado AES-GCM
│       ├── DataCipher.kt          # Cifrado de datos
│       ├── PasswordDeriver.kt     # Derivación de contraseñas
│       ├── SecureStorage.kt       # Almacenamiento seguro
│       └── BiometricAuthenticator.kt # Autenticación biométrica
│
├── domain/                        # Capa de Dominio
│   ├── model/                     # Modelos de negocio
│   ├── repository/                # Interfaces de repositorios
│   └── usecase/                   # Casos de uso
│       ├── auth/                  # Autenticación
│       ├── password/              # Gestión de contraseñas
│       ├── category/              # Gestión de categorías
│       ├── settings/              # Configuración
│       ├── backup/                # Copias de seguridad
│       ├── audit/                 # Auditoría de seguridad
│       └── statistics/            # Estadísticas
│
├── presentation/                  # Capa de Presentación
│   ├── ui/
│   │   ├── screens/               # Pantallas completas
│   │   ├── components/            # Componentes reutilizables
│   │   ├── state/                 # Estados de UI
│   │   └── viewmodel/             # ViewModels
│   ├── navigation/                # Grafo de navegación
│   ├── theme/                     # Tema y diseño
│   ├── widget/                    # Widgets de la app
│   ├── MainActivity.kt            # Actividad principal
│   └── AutoLockManager.kt         # Gestión de bloqueo automático
│
└── di/                            # Inyección de Dependencias
    └── AppModule.kt               # Módulo principal Koin
```

## Arquitectura

La aplicación sigue **Clean Architecture** con patrón **MVVM**:

```
┌─────────────────────────────────────────────────────────┐
│                   Presentation Layer                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │   Screens   │  │ ViewModels  │  │    UI State     │ │
│  └─────────────┘  └─────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                     Domain Layer                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │   Models    │  │  Use Cases  │  │  Repositories   │ │
│  └─────────────┘  └─────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                      Data Layer                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │   Entities  │  │   Repos     │  │   Local DB      │ │
│  └─────────────┘  └─────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

## Módulos de Documentación

| Módulo                                           | Descripción                              |
| ------------------------------------------------ | ---------------------------------------- |
| [Arquitectura](arquitectura/overview.md)         | Descripción detallada de la arquitectura |
| [Capa de Datos](data/overview.md)                | Persistencia, repositorios y seguridad   |
| [Capa de Dominio](domain/overview.md)            | Modelos y casos de uso                   |
| [Capa de Presentación](presentation/overview.md) | UI, ViewModels y navegación              |
| [Seguridad](security/overview.md)                | Sistema de cifrado y autenticación       |
| [Inyección de Dependencias](di/overview.md)      | Configuración de Koin                    |

## Configuración de Desarrollo

### Requisitos

* Android Studio Hedgehog o superior
* JDK 17
* Android SDK 26+ (mínimo), SDK 34 (objetivo)
* Gradle 8.0+

### Dependencias Principales

```kotlin
// Jetpack Compose + Material 3
androidx.compose.bom
androidx.compose.material3

// Room Database
androidx.room.runtime
androidx.room.ktx

// Koin (Inyección de Dependencias)
io.insert-koin:koin-core
io.insert-koin:koin-android

// Seguridad
androidx.security.crypto
androidx.biometric
```

## Estructura de la Base de Datos

### Tablas Principales

1. **PasswordEntry** - Almacena las entradas de contraseñas
2. **Category** - Categorías para organizar contraseñas
3. **Settings** - Configuración de la aplicación (clave-valor)

### Esquema de PasswordEntry

| Campo      | Tipo    | Descripción                          |
| ---------- | ------- | ------------------------------------ |
| id         | String  | Identificador único (UUID)           |
| title      | String  | Título de la entrada                 |
| username   | String  | Nombre de usuario                    |
| password   | String  | Contraseña cifrada (Base64)          |
| notes      | String? | Notas adicionales cifradas           |
| url        | String? | URL del sitio                        |
| categoryId | String  | FK a Category                        |
| icon       | String? | Icono de Font Awesome                |
| isFavorite | Boolean | Marcado como favorito                |
| createdAt  | Long    | Timestamp de creación (epoch ms)     |
| updatedAt  | Long    | Timestamp de modificación (epoch ms) |

### Esquema de Category

| Campo       | Tipo    | Descripción                 |
| ----------- | ------- | --------------------------- |
| id          | String  | Identificador único (UUID)  |
| name        | String  | Nombre de la categoría      |
| color       | Int     | Color ARGB (ej: 0xFFFF5722) |
| icon        | String  | Icono de Font Awesome       |
| isCustom    | Boolean | true = creada por usuario   |
| isDeletable | Boolean | false = predefinida         |

### Esquema de Settings

| Campo     | Tipo   | Descripción                |
| --------- | ------ | -------------------------- |
| key       | String | Clave de configuración     |
| value     | String | Valor almacenado           |
| updatedAt | Long   | Timestamp de actualización |

**Claves disponibles**:

* `theme_mode` - Modo de tema (0=Auto, 1=Light, 2=Dark)
* `biometric_enabled` - Biometría activada (true/false)
* `lock_timeout` - Timeout de auto-bloqueo en minutos

***

## Sistema de Seguridad

### Cifrado Híbrido

La aplicación utiliza un sistema de cifrado de doble capa:

```
┌─────────────────────────────────────────────────────────┐
│          Password Maestro del Usuario                    │
│                    (memoria)                             │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
            ┌─────────────────────────┐
            │  PBKDF2-HMAC-SHA256     │
            │  100,000 iteraciones    │
            │  Salt: 16 bytes         │
            └─────────────────────────┘
                          │
                          ▼
            ┌─────────────────────────┐
            │   Clave Derivada        │
            │   (256 bits / 32 bytes) │
            └─────────────────────────┘
                          │
          ┌───────────────┴───────────────┐
          ▼                               ▼
┌─────────────────────────┐   ┌─────────────────────────┐
│   RSA-2048 (Keystore)   │   │   AES-256-GCM           │
│   Cifra la clave        │   │   Cifra los datos       │
│   Requiere biometría    │   │   IV: 12 bytes          │
│   para descifrar        │   │   Tag: 128 bits         │
└─────────────────────────┘   └─────────────────────────┘
          │                               │
          ▼                               ▼
┌─────────────────────────┐   ┌─────────────────────────┐
│  SecureStorage          │   │  Room Database          │
│  - salt (Base64)        │   │  - password (cifrado)   │
│  - encrypted_key (RSA)  │   │  - notes (cifrado)      │
└─────────────────────────┘   └─────────────────────────┘
```

### Algoritmos Criptográficos

| Componente    | Algoritmo          | Parámetros                | Propósito                |
| ------------- | ------------------ | ------------------------- | ------------------------ |
| Derivación    | PBKDF2-HMAC-SHA256 | 100,000 iteraciones       | Crear clave del password |
| Cifrado Clave | RSA-2048-PKCS1     | Keystore hardware-backed  | Proteger clave maestra   |
| Cifrado Datos | AES-256-GCM        | IV 12 bytes, Tag 128 bits | Cifrar contraseñas       |
| Autenticación | BiometricPrompt    | BIOMETRIC\_STRONG         | Desbloquear Keystore     |

### Flujo de Autenticación

1. **Setup Inicial**:
   * Usuario crea password maestro
   * Se genera salt aleatorio (16 bytes)
   * Se deriva clave con PBKDF2 (100,000 iteraciones)
   * Se genera par RSA en Keystore
   * Se cifra clave derivada con RSA (clave pública)
   * Se guarda salt + clave cifrada en SecureStorage
2. **Login**:
   * Usuario ingresa password
   * Se deriva clave con el salt almacenado
   * Se solicita autenticación biométrica
   * Se descifra clave almacenada (RSA, clave privada)
   * Se comparan claves (derivada vs descifrada)
   * Si coinciden → sesión iniciada
3. **Cifrado de Datos**:
   * Cada password/nota se cifra individualmente
   * Se genera IV único por operación (12 bytes)
   * Se usa AES-256-GCM (confidencialidad + autenticidad)
   * Resultado: IV + ciphertext + tag

### Protección contra Ataques

| Ataque                | Mitigación                                |
| --------------------- | ----------------------------------------- |
| Fuerza bruta          | PBKDF2 100,000 iteraciones (lento)        |
| Rainbow tables        | Salt único aleatorio por usuario          |
| Extracción física     | AES-256-GCM con tag de autenticación      |
| Extracción de claves  | Android Keystore (hardware-backed)        |
| Replay attacks        | IV único aleatorio por operación          |
| Modificación de datos | GCM tag detecta alteraciones              |
| Shoulder surfing      | AutoLockManager (bloqueo por inactividad) |

***

## ViewModels Disponibles

| ViewModel                     | Responsabilidad            | Estados Clave             |
| ----------------------------- | -------------------------- | ------------------------- |
| `AuthViewModel`               | Login, setup, biometría    | `LoginState`              |
| `PasswordListViewModel`       | Lista, búsqueda, favoritos | `PasswordListState`       |
| `PasswordDetailViewModel`     | Detalle, copiado           | `PasswordDetailState`     |
| `PasswordFormViewModel`       | Crear/editar entrada       | `PasswordFormState`       |
| `PasswordGeneratorViewModel`  | Generar contraseña         | `PasswordGeneratorState`  |
| `CategoryManagementViewModel` | CRUD categorías            | `CategoryManagementState` |
| `SettingsViewModel`           | Configuración app          | `SettingsState`           |
| `BackupViewModel`             | Exportar/importar          | `BackupState`             |
| `AuditViewModel`              | Auditoría seguridad        | `AuditState`              |
| `StatisticsViewModel`         | Estadísticas               | `StatisticsState`         |
| `ChangePasswordViewModel`     | Cambiar password maestro   | `ChangePasswordState`     |

***

## Pantallas de la Aplicación

| Pantalla                   | Ruta                    | Descripción                          |
| -------------------------- | ----------------------- | ------------------------------------ |
| `LoginScreen`              | `/login`                | Autenticación con password/biometría |
| `OnboardingScreen`         | `/onboarding`           | Setup inicial (3 pasos)              |
| `HomeScreen`               | `/home`                 | Menú principal (grid 7 opciones)     |
| `PasswordListScreen`       | `/password_list`        | Lista con búsqueda y filtros         |
| `PasswordDetailScreen`     | `/password_detail/{id}` | Detalle con botones de copiado       |
| `PasswordFormScreen`       | `/password_form`        | Crear/editar entrada                 |
| `PasswordGeneratorScreen`  | `/password_generator`   | Generador con opciones               |
| `CategoryManagementScreen` | `/category_management`  | CRUD categorías                      |
| `SettingsScreen`           | `/settings`             | Configuración general                |
| `BackupScreen`             | `/backup`               | Exportar/importar datos              |
| `AuditScreen`              | `/audit`                | Detección de passwords débiles       |
| `StatisticsScreen`         | `/statistics`           | Métricas de seguridad                |
| `ChangePasswordScreen`     | `/change_password`      | Cambiar password maestro             |

***

**Última Actualización**: 2026-06-10\
**Versión del Proyecto**: 1.0
