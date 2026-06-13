# Resume Analyser API

## Descripción del Proyecto

**Resume Analyser** es una API REST construida con **Spring Boot 3.5.5** que utiliza inteligencia artificial avanzada para analizar y evaluar currículums (CV) de forma automática. Este sistema está diseñado con estándares de grado empresarial para proporcionar análisis detallados de compatibilidad con sistemas ATS (Applicant Tracking Systems) e identificar oportunidades de mejora en los currículums.

### Características Principales

- **Análisis ATS Avanzado**: Evalúa currículums contra estándares de ATS a nivel empresarial
- **Autenticación JWT**: Sistema seguro de autenticación basado en tokens JWT
- **Gestión de Usuarios**: Registro, login, recuperación de contraseña con OTP
- **Búsqueda de Empleos**: Integración con API de empleos para encontrar oportunidades relevantes
- **Extracción de Contenido**: Procesa archivos PDF, DOC, DOCX automáticamente
- **Evaluaciones Históricas**: Almacena y recupera informes previos de análisis

---

## APIs Externas Utilizadas

### 1. **Google Generative AI (Gemini)**
- **Propósito**: Análisis inteligente y evaluación de currículums
- **Función**:
  - Analiza el contenido extraído del CV contra requisitos de roles específicos
  - Proporciona puntuación ATS (0-100)
  - Genera recomendaciones personalizadas de mejora
  - Identifica fortalezas y debilidades del CV
  - Evaluación estricta basada en estándares de ATS empresariales
  
- **Configuración**: Se requiere `genKey` (API key de Google Generative AI)
- **Versión**: Google GenAI SDK v1.15.0

### 2. **Adzuna Job API**
- **Propósito**: Búsqueda de oportunidades laborales
- **Función**:
  - Busca empleos basados en los roles del usuario y ubicación
  - Retorna lista de trabajos disponibles relevantes al perfil
  - Filtra por ubicación (actualmente configurado para Tamil Nadu)
  
- **Endpoint**: `https://api.adzuna.com/v1/api/jobs/in/search/1`
- **Parámetros Requeridos**:
  - `app_id`: ID de aplicación de Adzuna
  - `app_key`: API key de Adzuna
  - `what`: Rol/palabras clave de búsqueda
  - `where`: Ubicación geográfica

### 3. **Servicio de Correo (SMTP)**
- **Propósito**: Envío de notificaciones y códigos OTP
- **Función**:
  - Envía códigos OTP para verificación de email
  - Notificaciones de recuperación de contraseña
  - Confirmaciones de registro de usuario
  
- **Configuración**: Spring Boot Mail Starter (SMTP)

---

## Endpoints de la API

### **Base URL**: `http://localhost:8080`

### **Rutas de Autenticación** (`resumeAnalyser/entry/v1`)

#### 1. Registro de Usuario
```
POST /resumeAnalyser/entry/v1/register
Content-Type: application/json

Body:
{
  "email": "usuario@example.com",
  "password": "contraseña",
  "username": "nombre_usuario"
}

Response: 201 Created
```

#### 2. Login
```
POST /resumeAnalyser/entry/v1/login
Content-Type: application/json

Body:
{
  "email": "usuario@example.com",
  "password": "contraseña"
}

Response: 200 OK
(Retorna JWT token en cookie)
```

#### 3. Verificar Email
```
POST /resumeAnalyser/entry/v1/verifyEmail
Content-Type: application/json

Body:
{
  "email": "usuario@example.com",
  "otp": "123456"
}

Response: 200 OK
```

#### 4. Solicitar OTP para Reset de Contraseña
```
POST /resumeAnalyser/entry/v1/resetOtpSent
Content-Type: application/json

Body:
{
  "email": "usuario@example.com"
}

Response: 200 OK
(Envía OTP al email)
```

#### 5. Verificar OTP de Reset
```
POST /resumeAnalyser/entry/v1/verifyResetOtp
Content-Type: application/json

Body:
{
  "email": "usuario@example.com",
  "otp": "123456"
}

Response: 200 OK
```

#### 6. Reset de Contraseña
```
POST /resumeAnalyser/entry/v1/resetPassword
Content-Type: application/json

Body:
{
  "email": "usuario@example.com",
  "newPassword": "nueva_contraseña"
}

Response: 200 OK
```

---

### **Rutas del Servicio Principal** (`resumeAnalyserCore/service/v1`)

#### 1. Análisis de CV (Endpoint Principal)
```
POST /resumeAnalyserCore/service/v1/extract
Content-Type: multipart/form-data

Parameters:
- file: Archivo del CV (PDF, DOC, DOCX)
- roles: Roles objetivo para evaluación (ej: "Java Developer, Data Scientist")

Response: 200 OK
{
  "score": 85,
  "atsOptimizationScore": 78,
  "pros": ["Buena experiencia técnica", "CV bien estructurado"],
  "cons": ["Faltan palabras clave relevantes"],
  "suggestions": ["Agregar certificaciones", "Mejorar descripción de proyectos"],
  "jobRecommendations": [...]
}
```

#### 2. Obtener Último Reporte
```
GET /resumeAnalyserCore/service/v1/lastReport
Authorization: Bearer <JWT_TOKEN>

Response: 200 OK
(Retorna el análisis más reciente del usuario)
```

#### 3. Validar Token
```
POST /resumeAnalyserCore/service/v1/isValid
Authorization: Bearer <JWT_TOKEN>

Response: 200 OK / 401 Unauthorized
```

#### 4. Logout
```
POST /resumeAnalyserCore/service/v1/logout
Authorization: Bearer <JWT_TOKEN>

Response: 200 OK
(Invalida el token JWT)
```

#### 5. Eliminar Cuenta
```
POST /resumeAnalyserCore/service/v1/deleteAccount
Authorization: Bearer <JWT_TOKEN>

Response: 200 OK
(Elimina la cuenta del usuario)
```

---

## Tecnologías Utilizadas

| Tecnología | Versión | Propósito |
|-----------|---------|----------|
| Spring Boot | 3.5.5 | Framework principal |
| Spring Security | - | Autenticación y autorización |
| Spring Data JPA | - | Gestión de base de datos |
| MySQL | - | Base de datos |
| JWT (JJWT) | 0.12.6 | Tokens de autenticación |
| Google GenAI | 1.15.0 | IA para análisis de CV |
| Apache Tika | - | Extracción de texto de documentos |
| Thymeleaf | - | Plantillas para emails |
| Lombok | - | Generación de código boilerplate |
| Maven | - | Gestor de dependencias |

---

## Configuración Requerida

### Variables de Entorno
```properties
# Base de Datos
spring.datasource.url=jdbc:mysql://localhost:3306/resume_db
spring.datasource.username=root
spring.datasource.password=tu_contraseña

# Google GenAI API
genKey=${GOOGLE_GENAI_API_KEY}

# Adzuna Job API
application-id=${ADZUNA_APP_ID}
application-api-key=${ADZUNA_APP_KEY}

# SMTP (para envío de emails)
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=${EMAIL_USERNAME}
spring.mail.password=${EMAIL_PASSWORD}
```

---

## Cómo Usar la API

### 1. Registrarse
```bash
curl -X POST http://localhost:8080/resumeAnalyser/entry/v1/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "usuario@example.com",
    "password": "segura123",
    "username": "usuario123"
  }'
```

### 2. Login
```bash
curl -X POST http://localhost:8080/resumeAnalyser/entry/v1/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "usuario@example.com",
    "password": "segura123"
  }'
```

### 3. Analizar CV
```bash
curl -X POST http://localhost:8080/resumeAnalyserCore/service/v1/extract \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -F "file=@mi_cv.pdf" \
  -F "roles=Java Developer"
```

---

## Estructura del Proyecto

```
src/main/java/com/ai/Resume/analyser/
├── controller/           # Controladores REST
│   ├── appController.java
│   ├── securityController.java
│   └── frontController.java
├── service/             # Lógica de negocio
│   ├── appService.java
│   ├── securityService.java
│   └── ...
├── model/               # Modelos de datos
│   ├── usersTable.java
│   ├── previousTable.java
│   └── ...
├── repository/          # Acceso a base de datos
├── jwt/                 # Configuración JWT
│   ├── jwtService.java
│   └── jwtFilter.java
└── config/              # Configuración de seguridad
```

---

## Notas Importantes

- **CORS**: Actualmente configurado para `http://localhost:5173` (frontend local)
- **Seguridad**: Utiliza JWT con tokens almacenados en cookies HttpOnly
- **OTP**: Sistema de verificación de dos pasos para seguridad
- **Base de Datos**: Requiere MySQL configurado
- **APIs Externas**: Requiere claves de API válidas para Google GenAI y Adzuna

---

## Autor

**Jose Navarro** - Desarrollador Full Stack

---

## Licencia

Este proyecto es de código abierto bajo la licencia que se especifique.

