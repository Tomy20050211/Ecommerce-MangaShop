# Ecommerce-MangaShop
# 📝 Examen Práctico — Desarrollo Full Stack
## MangaVerse: Tienda Online de Mangas

**Modalidad:** Individual  
**Tiempo estimado:** 5–8 días  
**Entrega:** Repositorio en GitHub con README de instalación  

---

## 📋 Contexto

Una empresa de entretenimiento japonés requiere una plataforma de e-commerce llamada **MangaVerse**, especializada en la venta de mangas físicos. Usted ha sido contratado como desarrollador Full Stack para construir dicha plataforma desde cero.

La tienda debe tener **catálogo obligatorio** con todos los tomos de las siguientes series al momento de iniciar la aplicación (vía seed):

| Serie | Autor | Tomos |
|-------|-------|-------|
| Dragon Ball | Akira Toriyama | 42 tomos |
| Fire Force | Atsushi Ohkubo | 34 tomos |
| The Future Diary | Sakae Esuno | 12 tomos |

> ⚠️ La ausencia de cualquiera de estas series en el catálogo inicial será causal de descuento en la nota.

---

## 🧰 Stack Obligatorio

El proyecto **debe** implementarse con las siguientes tecnologías. No se aceptan alternativas salvo indicación expresa del docente.

| Capa | Tecnología |
|------|-----------|
| Frontend | React + Vite + TypeScript |
| Estilos | Tailwind CSS |
| Backend | Node.js + Express |
| Base de datos | PostgreSQL |
| ORM | Prisma |
| Autenticación JWT | JOSE (librería `jose`) |
| Hash de contraseñas | Bcrypt (`bcryptjs`) |
| Validación de esquemas | Zod |
| Estado global (cliente) | Zustand |
| Peticiones HTTP | Axios |

---

## 🔐 Punto 1 — Sistema de Autenticación Segura (25%)

Implemente un sistema de autenticación completo que cumpla los siguientes requisitos:

### 1.1 Registro e Inicio de Sesión

- El usuario debe poder **registrarse** proporcionando: nombre de usuario, correo electrónico y contraseña.
- La contraseña debe ser **hasheada con Bcrypt** antes de almacenarse. Nunca se debe guardar en texto plano.
- Al iniciar sesión con credenciales correctas, el servidor debe generar y firmar un **JSON Web Token usando la librería JOSE** (algoritmo HS256).

### 1.2 Cookies Seguras

- El JWT generado debe ser enviado al cliente **exclusivamente mediante una cookie HTTP** con las siguientes propiedades obligatorias:
  - `httpOnly: true` — La cookie no debe ser accesible desde JavaScript del lado del cliente.
  - `secure: true` — Solo se debe enviar por HTTPS (en producción).
  - `sameSite: 'strict'` — Para mitigar ataques CSRF.
  - Expiración de **7 días**.
- Queda **prohibido** retornar el token en el cuerpo de la respuesta (`res.json({ token })`).

### 1.3 Middleware de Verificación

- Cree un middleware que se ejecute en cada ruta protegida. Dicho middleware debe:
  1. Leer el token desde la cookie de la petición entrante.
  2. Verificar y decodificar el token usando JOSE.
  3. Adjuntar los datos del usuario (`id`, `email`, `role`) al objeto `req` para uso posterior.
  4. Retornar `401 Unauthorized` si el token no existe, es inválido o ha expirado.

### 1.4 Validación de Roles

- El sistema debe manejar dos roles: `USER` y `ADMIN`.
- Implemente un **guard de roles** reutilizable que reciba el rol requerido como parámetro y retorne `403 Forbidden` si el usuario autenticado no posee dicho rol.
- Todas las rutas del panel de administración deben estar protegidas por este guard.

### 1.5 Validación de Datos de Entrada

- Use **Zod** para validar los cuerpos de las peticiones de registro y login antes de procesarlos.
- El esquema de contraseña debe exigir: mínimo 8 caracteres, al menos una mayúscula y al menos un número.
- Retorne errores descriptivos al cliente si la validación falla (`400 Bad Request`).

### 1.6 Cierre de Sesión

- Implemente un endpoint `POST /auth/logout` que elimine la cookie del cliente correctamente.

---

## 👤 Punto 2 — Dashboard Interactivo de Usuario (25%)

Una vez autenticado, el usuario con rol `USER` debe tener acceso a un panel con las siguientes funcionalidades:

### 2.1 Catálogo de Mangas

- Mostrar todos los mangas disponibles en formato de **tarjetas (cards)**, cada una con: imagen de portada, título, serie, número de tomo, precio y estado.
- Los mangas con estado `COMING_SOON` o `OUT_OF_STOCK` deben mostrarse visualmente diferenciados (opacidad reducida, badge de estado) y **no permitir ser agregados al carrito**.

### 2.2 Buscador

- Implementar un campo de búsqueda que filtre los mangas **en tiempo real** (debounce recomendado) por: título, serie o autor.
- La búsqueda debe realizarse mediante una petición al backend, no solo en el estado local del cliente.

### 2.3 Panel de Filtros

El catálogo debe poder filtrarse por los siguientes criterios (pueden combinarse):

- **Serie:** Dragon Ball / Fire Force / The Future Diary / Todas.
- **Estado:** Disponible / Agotado / Próximamente.
- **Rango de precio:** mediante un slider o inputs numéricos (mín. – máx.).
- **Ordenar por:** precio ascendente, precio descendente, nombre A-Z, más recientes.

### 2.4 Carrito de Compras

- El usuario debe poder **agregar tomos al carrito** desde el catálogo.
- El carrito debe persistir en base de datos (no solo en localStorage).
- Debe ser posible **modificar la cantidad** de un tomo o **eliminarlo** del carrito.
- El carrito debe mostrar el **subtotal** actualizado en tiempo real.
- No se debe poder agregar al carrito un tomo con estado diferente a `AVAILABLE`.

### 2.5 Proceso de Compra Ficticia

- El usuario puede finalizar la compra desde el carrito mediante un botón **"Confirmar compra"**.
- El sistema debe:
  1. Verificar que todos los tomos en el carrito sigan disponibles.
  2. Calcular el total (aplicando descuentos activos si los hay).
  3. Registrar la compra en la base de datos como una `Order` con sus `OrderItem`.
  4. Vaciar el carrito del usuario tras la compra exitosa.
  5. Mostrar al usuario un resumen de la orden completada.

### 2.6 Historial de Compras

- El usuario debe poder consultar **todas sus órdenes anteriores**, incluyendo: fecha, tomos comprados (con portada en miniatura), cantidad y total pagado.

### 2.7 EXTRA — Coincidencias entre Mangas (+5%)

- En la vista de detalle de un manga, mostrar una sección **"También te puede interesar"** con hasta 6 mangas relacionados, calculados en base a:
  - Misma serie (otros tomos).
  - Mismo autor.
  - Géneros en común.
- Este cálculo debe hacerse en el backend.

---

## 👑 Punto 3 — Dashboard Interactivo de Administrador (25%)

El usuario con rol `ADMIN` debe tener acceso a un panel de administración completamente separado del panel de usuario, con las siguientes funcionalidades:

### 3.1 CRUD Completo de Mangas

El administrador debe poder realizar las siguientes operaciones sobre el catálogo:

- **Listar** todos los mangas con paginación (mínimo 20 por página).
- **Crear** un nuevo tomo mediante un formulario.
- **Editar** la información de un tomo existente.
- **Eliminar** un tomo del catálogo (con confirmación previa).

### 3.2 Formulario de Creación / Edición de Tomo

El formulario debe incluir y validar los siguientes campos:

| Campo | Tipo | Validación |
|-------|------|-----------|
| Título | Texto | Requerido |
| Serie | Select | Requerido |
| Número de volumen | Número | Requerido, entero positivo |
| Autor | Texto | Requerido |
| Sinopsis | Textarea | Requerido, mín. 20 caracteres |
| Precio | Número | Requerido, mayor a 0 |
| Géneros | Multi-select | Al menos uno |
| Estado | Select | Requerido |
| Imagen de portada | Archivo | JPG/PNG/WEBP, máx. 5MB |

- La imagen debe subirse al servidor y su URL debe almacenarse en la base de datos.
- Use `multer` en el backend para gestionar la subida de archivos.

### 3.3 Gestión de Estados por Tomo

- El administrador debe poder cambiar el estado de cualquier tomo directamente desde la tabla del catálogo, sin necesidad de abrir el formulario de edición completo.
- Los estados disponibles son:
  - 🟢 `AVAILABLE` — Disponible para compra.
  - 🔴 `OUT_OF_STOCK` — Agotado temporalmente.
  - 🟡 `COMING_SOON` — Próximamente disponible.

### 3.4 Panel de Estadísticas de Ventas

El dashboard del administrador debe incluir un panel de estadísticas que muestre:

- **Total de órdenes** registradas en la plataforma.
- **Ingresos totales** generados.
- **Ranking de los 5 tomos más vendidos**, mostrando: título, número de unidades vendidas e ingresos generados.
- **Gráfico de ventas** por serie (puede ser de barras o de torta). Implemente usando la librería `recharts`.
- **Últimas 10 órdenes** realizadas con: usuario, monto y fecha.

> Para cada tomo en el catálogo debe ser visible cuántas veces ha sido comprado.

### 3.5 EXTRA — Sistema de Descuentos (+5%)

- El administrador debe poder aplicar un **descuento porcentual** a cualquier tomo del catálogo.
- El descuento debe tener los siguientes atributos: porcentaje (ej. `15` para 15%), estado activo/inactivo y fecha de expiración opcional.
- Un descuento activo debe reflejarse visualmente en el catálogo del usuario (precio tachado + precio con descuento + badge `"-15%"`).
- El descuento debe aplicarse automáticamente al calcular el total de una compra.
- Si un descuento tiene fecha de expiración y esta ya pasó, debe ignorarse aunque esté marcado como activo.

---

## 🗄️ Punto 4 — Modelo de Datos y Seed (10%)

### 4.1 Esquema de Base de Datos

Diseñe e implemente en **Prisma** el esquema relacional que soporte todas las funcionalidades anteriores. Como mínimo debe contemplar las siguientes entidades: `User`, `Manga`, `Order`, `OrderItem`, `Cart`, `CartItem`. El esquema será evaluado en términos de normalización, relaciones correctas y uso de tipos adecuados.

### 4.2 Script de Seed

- Cree un script `prisma/seed.ts` que al ejecutarse con `npx prisma db seed` pueble la base de datos con **todos los tomos obligatorios**:
  - 42 tomos de Dragon Ball.
  - 34 tomos de Fire Force.
  - 12 tomos de The Future Diary.
- Cada tomo debe tener datos coherentes (autor correcto, géneros apropiados, precio, estado `AVAILABLE` por defecto).
- El script debe ser **idempotente**: ejecutarlo múltiples veces no debe duplicar registros.

---

## 🔒 Punto 5 — Calidad y Seguridad del Código (15%)

### 5.1 Seguridad General

- Configure **Helmet** en el servidor Express para establecer cabeceras HTTP de seguridad.
- Implemente **rate limiting** en las rutas `/auth/login` y `/auth/register` (máximo 10 intentos cada 15 minutos por IP).
- Configure **CORS** correctamente para aceptar peticiones solo del origen del frontend, con soporte de credenciales.

### 5.2 Tipado y Validación

- Todo el código backend y frontend debe estar escrito en **TypeScript** sin uso de `any` explícito.
- Todos los cuerpos de petición en el backend deben ser validados con **Zod** antes de procesarse.

### 5.3 Manejo de Errores

- El servidor debe retornar respuestas de error consistentes en formato JSON: `{ error: string, details?: any }`.
- No deben exponerse stack traces ni mensajes internos al cliente en producción.

### 5.4 Testing

- Escriba al menos **4 pruebas** con Supertest que cubran:
  1. Registro exitoso de usuario.
  2. Login con credenciales incorrectas (debe retornar 401).
  3. Acceso a ruta de admin con rol USER (debe retornar 403).
  4. Agregar un manga al carrito sin estar autenticado (debe retornar 401).

---

## 📁 Entregables

El repositorio de GitHub debe contener:

1. Código fuente completo (backend y frontend en carpetas separadas).
2. Archivo `README.md` con instrucciones claras para:
   - Clonar el repositorio.
   - Configurar variables de entorno (incluir `.env.example`).
   - Ejecutar migraciones y seed.
   - Levantar el proyecto en modo desarrollo.
3. Archivo `.env.example` con todas las variables necesarias (sin valores reales).
4. El schema de Prisma con migraciones generadas (`prisma/migrations/`).

---

## ⚖️ Tabla de Evaluación

| Criterio | Puntaje |
|----------|---------|
| Punto 1 — Autenticación (JWS + Bcrypt + Cookies + Roles) | 25% |
| Punto 2 — Dashboard de Usuario | 25% |
| Punto 3 — Dashboard de Administrador | 25% |
| Punto 4 — Modelo de datos y Seed obligatorio | 10% |
| Punto 5 — Calidad, seguridad y testing | 15% |
| **EXTRA** — Coincidencias entre mangas | +5% |
| **EXTRA** — Sistema de descuentos | +5% |
| **TOTAL** | **100% (+10%)** |

---

## ⚠️ Notas Importantes

- El uso de librerías de autenticación de alto nivel como `passport`, `next-auth` o similares está **prohibido**. La autenticación debe implementarse manualmente con JOSE y Bcrypt según lo especificado.
- Queda prohibido almacenar el JWT en `localStorage` o `sessionStorage`.
- Las rutas del panel de administrador deben estar protegidas tanto en el **backend** (middleware + guard) como en el **frontend** (ruta privada con verificación de rol).
- El proyecto debe poder levantarse localmente siguiendo únicamente las instrucciones del README. Un proyecto que no levante no será evaluado.
- Se penalizará con hasta un **20%** el uso de código generado íntegramente por IA sin comprensión demostrable del mismo.
