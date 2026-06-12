# Password Manager - Documentación del Proyecto

## Visión General

Password Manager es una aplicación Android nativa para la gestión segura de contraseñas, desarrollada con **Jetpack Compose** y siguiendo arquitecturas modernas de desarrollo Android.

### Características Principales

- **Almacenamiento Seguro**: Las contraseñas se cifran utilizando AES-GCM a través de Android Keystore
- **Autenticación Biométrica**: Soporte para huella dactilar y reconocimiento facial
- **Gestión de Categorías**: Organización de contraseñas por categorías personalizadas
- **Generador de Contraseñas**: Creación de contraseñas seguras con opciones configurables
- **Búsqueda y Filtrado**: Búsqueda en tiempo real y filtrado por categorías
- **Favoritos**: Marcado de contraseñas frecuentes para acceso rápido
- **Copia de Seguridad**: Exportación e importación de datos cifrados
- **Auditoría de Seguridad**: Detección de contraseñas débiles
- **Estadísticas**: Resumen del estado de seguridad de las contraseñas

## Stack Tecnológico

| Componente | Tecnología |
|------------|------------|
| **UI Framework** | Jetpack Compose |
| **Arquitectura** | Clean Architecture + MVVM |
| **Base de Datos** | Room Database |
| **Inyección de Dependencias** | Koin |
| **Cifrado** | Android Keystore + AES-GCM |
| **Navegación** | Navigation Compose |
| **Preferencias** | DataStore |
| **Serialización** | Kotlinx Serialization |

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

| Módulo | Descripción |
|--------|-------------|
| [Arquitectura](arquitectura/overview.md) | Descripción detallada de la arquitectura |
| [Capa de Datos](data/overview.md) | Persistencia, repositorios y seguridad |
| [Capa de Dominio](domain/overview.md) | Modelos y casos de uso |
| [Capa de Presentación](presentation/overview.md) | UI, ViewModels y navegación |
| [Seguridad](security/overview.md) | Sistema de cifrado y autenticación |
| [Inyección de Dependencias](di/overview.md) | Configuración de Koin |

## Configuración de Desarrollo

### Requisitos

- Android Studio Hedgehog o superior
- JDK 17
- Android SDK 26+ (mínimo), SDK 34 (objetivo)
- Gradle 8.0+

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
3. **Settings** - Configuración de la aplicación

### Esquema de PasswordEntry

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | String | Identificador único (UUID) |
| title | String | Título de la entrada |
| username | String | Nombre de usuario |
| password | String | Contraseña cifrada (Base64) |
| notes | String? | Notas adicionales cifradas |
| url | String? | URL del sitio |
| categoryId | String | FK a Category |
| createdAt | Date | Fecha de creación |
| updatedAt | Date | Fecha de modificación |
| isFavorite | Boolean | Marcado como favorito |

## Índice Completo

Consultar [SUMMARY.md](SUMMARY.md) para el índice completo de toda la documentación.

---

**Última Actualización**: 2026-06-10  
**Versión del Proyecto**: 1.0
