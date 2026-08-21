# Trade Management System (TSM)

Sistema de gestión comercial **multi-tenant** pensado para negocios con una o varias sucursales: control de ventas, compras, stock, caja, clientes, proveedores y equipo de trabajo, con estadísticas y permisos por rol.

Full-stack propio: **Spring Boot** en el backend y **Angular 18** en el frontend, con un sistema de diseño dark consistente en toda la aplicación.

---

## Tabla de contenidos

- [Características principales](#características-principales)
- [Stack tecnológico](#stack-tecnológico)
- [Arquitectura](#arquitectura)
- [Roles y permisos](#roles-y-permisos)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Repositorios](#repositories)
- [Puesta en marcha](#puesta-en-marcha)
- [Roadmap](#roadmap)

---

## Características principales

### 🏢 Multi-tenant / Multi-sucursal
- Un usuario puede administrar **varios comercios**, cada uno con sus propias sucursales, productos, equipo y datos independientes.
- Selector de comercio y sucursal activa persistido entre sesiones.

### 👥 Equipo y permisos (RBAC)
- Tres roles: **OWNER**, **ADMIN** y **EMPLOYEE**, con matriz de acceso granular tanto en el backend (validaciones por endpoint) como en el frontend (guards de ruta + nav dinámico).
- Los empleados pueden restringirse a una o varias sucursales específicas.
- Invitación de usuarios existentes por email a un comercio, con selección de rol y sucursal.
- Registro de un usuario nuevo directamente desde la pantalla de equipo, sin perder la sesión activa (antes requería cerrar sesión, registrar al usuario e iniciar sesión de nuevo).

### 📦 Productos
- Productos con **variantes** (talles, colores, presentaciones, etc.).
- Categorías y **proveedor asociado** por producto, para poder filtrar qué se le compra a quién.
- **Actualización masiva de precios**: aplicá un aumento o descuento porcentual a todos los productos de un proveedor de una sola vez (precio de venta, costo, o ambos).

### 🛒 Ventas y Compras
- Carrito de compras/ventas con cantidad **editable manualmente**, además de los botones +/-.
- El formulario de compra filtra los productos por proveedor seleccionado: primero elegís el proveedor y solo ves lo que le comprás a él.
- Distintas formas de pago: efectivo, tarjeta, transferencia, mixto y cuenta corriente.
- Cuentas corrientes de clientes y proveedores, con historial de deuda y liquidación de pagos.

### 💵 Caja
- Apertura y cierre de caja por sucursal, con registro de movimientos.
- Historial de aperturas/cierres con **filtros de fecha** (desde/hasta) y balance de cada ciclo.

### 📊 Estadísticas (solo OWNER)
Panel exclusivo para el dueño del comercio con:
- **Ventas del mes actual vs. mes anterior**, con variación porcentual y evolución de los últimos 12 meses (gráfico de línea).
- **Producto más vendido**, top 5 por cantidad y por dinero generado.
- **Ventas por sucursal**, para comparar rendimiento entre locales.
- **Ventas por forma de pago** (efectivo / tarjeta / cuenta corriente / etc.), gráfico de dona.
- **Alertas de stock bajo**, para anticipar quiebres de inventario.

### 📉 Stock
- Control de stock por sucursal y variante de producto, con umbral mínimo configurable y alertas de stock bajo.

### 🔐 Autenticación
- Login, registro, recuperación de contraseña con código de verificación, y reset de contraseña — todas con validaciones de email/contraseña y visor de contraseña (👁).
- Autenticación stateless con JWT.

---

## Stack tecnológico

**Backend**
- Java + Spring Boot
- Spring Data JPA / Hibernate
- Spring Security + JWT
- Bean Validation (Jakarta Validation)
- Lombok

**Frontend**
- Angular 18 (standalone components, signals, control flow `@if`/`@for`)
- Tailwind CSS v4
- Diseño dark consistente: fondos `#0f0f0f` / `#1a1a1a`, acento `indigo-500`, tipografía DM Sans
- Gráficos construidos en SVG puro (sin librerías externas de charting)

---

## Arquitectura

```
Commerce (comercio)
 ├── Branch (sucursal) — 1 a N por comercio
 │     ├── Stock (por variante de producto)
 │     ├── CashRegister (aperturas/cierres)
 │     ├── Sale (ventas)
 │     └── Purchase (compras)
 ├── Product ── ProductVariant (N variantes por producto)
 │     └── Supplier (proveedor asociado, opcional)
 ├── Supplier / Customer (con cuenta corriente / deuda)
 └── UserBranch (asignación de empleados a sucursales)
```

- **Autorización en dos capas**: guards de Angular (`ownerGuard`, `adminGuard`) para la navegación, y validaciones equivalentes en cada endpoint del backend (`AuthorizationService`) — la UI nunca es la única barrera de seguridad.
- **Estado global del frontend** con signals (`AppStateService` para comercio/sucursal activos, `RoleService` para el rol del usuario en el comercio activo), persistido en `localStorage`.

---

## Roles y permisos

| Módulo                     | OWNER | ADMIN | EMPLOYEE |
|-----------------------------|:-----:|:-----:|:--------:|
| Comercios (crear/editar)    | ✅    | ❌    | ❌       |
| Equipo (invitar/registrar)  | ✅    | ❌    | ❌       |
| Estadísticas                | ✅    | ❌    | ❌       |
| Productos (crear/editar)    | ✅    | ✅    | ❌       |
| Compras                     | ✅    | ✅    | ❌       |
| Proveedores                 | ✅    | ✅    | ❌       |
| Ventas                      | ✅    | ✅    | ✅ *     |
| Clientes                    | ✅    | ✅    | ✅ *     |
| Stock (consulta)            | ✅    | ✅    | ✅ *     |
| Caja                        | ✅    | ✅    | ✅ *     |

\* Los empleados solo operan sobre las sucursales que tengan asignadas.

---

## Estructura del proyecto

```
trade-management-system/
├── back/
│   └── Trade-Management-System-Back/
│       └── src/main/java/com/tsm/api/
│           ├── controller/      # Endpoints REST
│           ├── service/         # Lógica de negocio (interfaces + impl)
│           ├── repository/      # Spring Data JPA + proyecciones
│           ├── entity/          # Entidades JPA
│           ├── dto/
│           │   ├── request/
│           │   └── response/
│           └── security/        # JWT, autorización por rol
│
└── front/
    └── Trade-Management-System-Front/front-app/
        └── src/app/
            ├── core/
            │   ├── services/    # Estado global, auth, servicios por dominio
            │   ├── guards/      # ownerGuard, adminGuard, authGuard
            │   ├── models/
            │   └── interceptors/ # JWT, manejo de errores 401/403
            └── features/
                ├── auth/
                ├── dashboard/   # Shell + navegación lateral
                ├── commerces/ · branches/ · team/
                ├── products/ · stock/
                ├── sales/ · purchases/
                ├── customers/ · suppliers/
                ├── cash-register/
                └── statistics/
```

---

## repositories

📁 Repository: [Trade-Management-System-Front](https://github.com/TomasHerrado/Trade-Management-System-Front.git)
📁 Repository: [Trade-Management-System](https://github.com/TomasHerrado/Trade-Management-System.git)

---

## Puesta en marcha

### Requisitos previos
- Java 17+ y Maven (o el wrapper `mvnw` incluido)
- Node.js 18+ y Angular CLI (`npm install -g @angular/cli`)
- Una base de datos relacional (PostgreSQL recomendado)

### Backend

```bash
cd back/Trade-Management-System-Back
# configurar src/main/resources/application.properties (ver más abajo)
./mvnw spring-boot:run
```
La API queda disponible en `http://localhost:8080/api`.

> El proyecto usa `spring.jpa.hibernate.ddl-auto=update`, así que el esquema se crea/actualiza solo al levantar la app. No hace falta correr migraciones a mano.

### Frontend

```bash
cd front/Trade-Management-System-Front/front-app
npm install
ng serve
```
La app queda disponible en `http://localhost:4200`.

---

## Roadmap

- [ ] Deuda total por comercio (clientes y proveedores) como estadística
- [ ] Reportes exportables (PDF/Excel) de ventas y compras
- [ ] Notificaciones automáticas de stock bajo
- [ ] Panel comparativo entre comercios para usuarios OWNER con más de uno
- [ ] Tests automatizados (backend e2e / frontend unit)

---

## Licencia

Proyecto privado / uso personal. Ajustar esta sección si se decide publicar bajo una licencia open source (MIT, Apache 2.0, etc.).
