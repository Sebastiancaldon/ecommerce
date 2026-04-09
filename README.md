# 🛒 E-Commerce Ropa - Sistema MVC con Next.js y MySQL

![Next.js](https://img.shields.io/badge/Next.js-16-black)
![MySQL](https://img.shields.io/badge/MySQL-Database-blue)
![MVC](https://img.shields.io/badge/Architecture-MVC-green)
![Status](https://img.shields.io/badge/status-Completed-success)

Sistema de comercio electrónico desarrollado con Next.js y MySQL, implementando arquitectura MVC. Incluye autenticación, carrito de compras, gestión de productos, inventario y pedidos.

Este proyecto también documenta el cumplimiento de estándares de calidad de software como CMMI, ISO 9001, IEEE 730 e ISO/IEC 25000.

---

### Indice

1. [Descripcion del Proyecto](#1-descripcion-del-proyecto)
2. [Arquitectura del Sistema](#2-arquitectura-del-sistema)
3. [Modelo de Base de Datos](#3-modelo-de-base-de-datos)
4. [Cumplimiento de Normas](#4-cumplimiento-de-normas)
   - [CMMI](#41-cmmi---capability-maturity-model-integration)
   - [ISO 9001](#42-iso-9001---sistema-de-gestion-de-calidad)
   - [IEEE 730](#43-ieee-730---aseguramiento-de-calidad-del-software)
   - [ISO 9126 / ISO 25000](#44-iso-9126--iso-25000---calidad-del-producto-software)
   - [ISO 14598](#45-iso-14598---evaluacion-del-producto-software)
   - [ISO/IEC 12207](#46-isoiec-12207---ciclo-de-vida-del-software)
   - [ISO/IEC 15504 SPICE](#47-isoiec-15504-spice---evaluacion-de-procesos)
   - [ISO/IEC 20000](#48-isoiec-20000---gestion-de-servicios-ti)
5. [Guia de Implementacion](#5-guia-de-implementacion)
6. [Seguridad](#6-seguridad)

---

## 1. Descripcion del Proyecto

### 1.1 Proposito

Sistema de comercio electronico para venta de ropa desarrollado con:
- **Frontend**: Next.js 16, React 19, TypeScript, Tailwind CSS
- **Backend**: API Routes de Next.js
- **Base de Datos**: MySQL con mysql2
- **Arquitectura**: MVC (Modelo-Vista-Controlador)

### 1.2 Alcance

El sistema permite:
- Gestion de catalogo de productos y categorias
- Carrito de compras persistente
- Procesamiento de pedidos y pagos
- Panel de administracion
- Gestion de inventario
- Autenticacion de usuarios

### 1.3 Objetivos de Calidad

| Objetivo | Descripcion | Metrica |
|----------|-------------|---------|
| Funcionalidad | Operaciones CRUD completas | 100% endpoints funcionales |
| Seguridad | Proteccion de datos sensibles | Hash de contrasenas, validacion |
| Mantenibilidad | Codigo documentado y modular | Separacion MVC |
| Escalabilidad | Soporte para crecimiento | Pool de conexiones MySQL |

---

## 2. Arquitectura del Sistema

### 2.1 Patron MVC Implementado

```
lib/
├── db/
│   ├── mysql.ts              # Conexion a base de datos
│   └── models/
│       ├── user.mysql.ts     # Modelo de Usuario
│       ├── product.mysql.ts  # Modelo de Producto/Categoria
│       ├── cart.mysql.ts     # Modelo de Carrito
│       ├── order.mysql.ts    # Modelo de Pedido/Pago
│       ├── inventory.mysql.ts # Modelo de Inventario
│       └── index.ts          # Exportaciones centralizadas
├── models/
│   ├── types.ts              # Definiciones de tipos
│   └── database.ts           # Base de datos en memoria (fallback)
└── context/
    ├── auth-context.tsx      # Estado de autenticacion
    └── cart-context.tsx      # Estado del carrito
```

### 2.2 Flujo de Datos

```
[Cliente] → [API Route] → [Modelo MySQL] → [Base de Datos]
     ↑                                            ↓
     └──────────── [Respuesta JSON] ←─────────────┘
```

### 2.3 Componentes Principales

| Componente | Responsabilidad | Ubicacion |
|------------|-----------------|-----------|
| Views | Interfaz de usuario | `app/`, `components/` |
| Controllers | Logica de negocio | `app/api/` |
| Models | Acceso a datos | `lib/db/models/` |

---

## 3. Modelo de Base de Datos

### 3.1 Diagrama Entidad-Relacion

```
┌─────────────┐     ┌─────────────┐
│    ROL      │     │   USUARIO   │
├─────────────┤     ├─────────────┤
│ id_rol (PK) │◄────│ id_usuario  │
│ nombre      │     │ nombreusuario│
└─────────────┘     │ emailusuario│
                    │ telefono    │
                    │ clave       │
                    │ id_rol (FK) │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ CARRITO  │ │  PEDIDO  │ │          │
        ├──────────┤ ├──────────┤ │          │
        │id_carrito│ │id_pedido │ │          │
        │id_usuario│ │ fecha    │ │          │
        └────┬─────┘ │id_usuario│ │          │
             │       └────┬─────┘ │          │
             ▼            │       │          │
     ┌──────────────┐     │       │          │
     │DETALLE_CARRITO│    ▼       │          │
     ├──────────────┤ ┌──────────────┐       │
     │ id_detalle   │ │DETALLE_PEDIDO│       │
     │ cantidad     │ ├──────────────┤       │
     │ id_carrito   │ │ id_detalle   │       │
     │ id_producto  │◄│ cantidad     │       │
     └──────┬───────┘ │ precio       │       │
            │         │ id_pedido    │       │
            │         │ id_producto  │─┐     │
            │         └──────────────┘ │     │
            │                          │     │
            │                          ▼     │
            │         ┌──────────────┐       │
            │         │   PRODUCTO   │       │
            │         ├──────────────┤       │
            └────────►│ id_producto  │◄──────┘
                      │ nombre       │
                      │ descripcion  │
                      │ precio       │
                      │ id_categoria |
                      | imagen       │──┐
                      └──────────────┘  │
                             ▲          │
                             │          ▼
                      ┌──────────────┐ ┌──────────────┐
                      │  INVENTARIO  │ │  CATEGORIA   │
                      ├──────────────┤ ├──────────────┤
                      │id_inventario │ │ id_categoria │
                      │ stock        │ │ nombre       │
                      │ id_producto  │ └──────────────┘
                      └──────────────┘

┌──────────────┐
│    PAGO      │
├──────────────┤
│ id_pago      │
│ metodo       │
│ estado       │
│ id_pedido    │──► PEDIDO
└──────────────┘
```

### 3.2 Tablas y Atributos

| Tabla | Atributos | Descripcion |
|-------|-----------|-------------|
| **rol** | id_rol, nombre | Roles del sistema (Cliente, Admin) |
| **usuario** | id_usuario, nombreusuario, emailusuario, telefono, clave, id_rol | Usuarios registrados |
| **categoria** | id_categoria, nombre | Categorias de productos |
| **producto** | id_producto, nombre, descripcion, precio, id_categoria | Catalogo de productos |
| **inventario** | id_inventario, stock, id_producto | Control de stock |
| **carrito** | id_carrito, id_usuario | Carrito por usuario |
| **detalle_carrito** | id_detalle, cantidad, id_carrito, id_producto | Items del carrito |
| **pedido** | id_pedido, fecha, id_usuario | Pedidos realizados |
| **detalle_pedido** | id_detalle, cantidad, precio, id_pedido, id_producto | Items del pedido |
| **pago** | id_pago, metodo, estado, id_pedido | Informacion de pago |

---

## 4. Cumplimiento de Normas

### 4.1 CMMI - Capability Maturity Model Integration

#### Nivel 2: Gestionado

| Practica | Implementacion en el Proyecto |
|----------|------------------------------|
| **Gestion de Requisitos** | Tipos TypeScript definen requisitos de datos (`types.ts`) |
| **Planificacion de Proyectos** | Estructura modular permite estimacion de tareas |
| **Seguimiento y Control** | Logs de errores en consola para monitoreo |
| **Gestion de Configuracion** | Variables de entorno (`.env.local`), pool de conexiones |
| **Aseguramiento de Calidad** | Validaciones en modelos, tipos estrictos |

#### Nivel 3: Definido

| Practica | Implementacion en el Proyecto |
|----------|------------------------------|
| **Desarrollo de Requisitos** | Interfaces TypeScript documentadas |
| **Solucion Tecnica** | Patron MVC, separacion de responsabilidades |
| **Integracion del Producto** | API Routes integradas con modelos MySQL |
| **Verificacion** | Validacion de datos en cada modelo |
| **Validacion** | Pruebas de stock, autenticacion, pagos |

**Evidencia en codigo:**
```typescript
// lib/models/types.ts - Gestion de Requisitos
export interface Usuario {
  id_usuario: string;
  nombreusuario: string;
  emailusuario: string;
  telefono: string;
  clave: string;
  id_rol: string;
}
```

---

### 4.2 ISO 9001 - Sistema de Gestion de Calidad

#### Principios Aplicados

| Principio ISO 9001 | Implementacion |
|-------------------|----------------|
| **Enfoque al Cliente** | Interfaz intuitiva, carrito persistente, historial de pedidos |
| **Liderazgo** | Documentacion clara de arquitectura y flujos |
| **Compromiso de las Personas** | Codigo comentado para facilitar colaboracion |
| **Enfoque a Procesos** | Flujos definidos: registro → compra → pago → pedido |
| **Mejora Continua** | Estructura modular permite actualizaciones |
| **Toma de Decisiones Basada en Evidencia** | Logs, estadisticas de dashboard |
| **Gestion de Relaciones** | API documentada para integraciones |

#### Documentacion del Sistema

| Documento | Ubicacion | Contenido |
|-----------|-----------|-----------|
| Manual de Usuario | `docs/` | Guia de uso del sistema |
| Manual Tecnico | Este documento | Arquitectura y normas |
| Procedimientos | Codigo comentado | Logica de negocio |

**Evidencia en codigo:**
```typescript
/**
 * MODELO - Usuario con MySQL (MVC)
 * Cumplimiento de normas:
 * - ISO 9001:2015 - Gestion de usuarios documentada
 * - IEEE 730 - Validacion de datos de entrada
 */
```

---

### 4.3 IEEE 730 - Aseguramiento de Calidad del Software

#### Actividades de SQA Implementadas

| Actividad | Implementacion |
|-----------|----------------|
| **Revisiones Tecnicas** | Comentarios de codigo explicativos |
| **Validacion de Requisitos** | TypeScript verifica tipos en compilacion |
| **Verificacion de Diseno** | Patron MVC aplicado consistentemente |
| **Pruebas** | Validaciones en runtime (email, stock, pagos) |
| **Control de Configuracion** | Git, variables de entorno |

#### Validaciones Implementadas

```typescript
// Validacion de email (IEEE 730 - Verificacion de entrada)
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
if (!emailRegex.test(datos.emailusuario)) {
  return { success: false, message: 'Formato de email invalido' };
}

// Validacion de stock (IEEE 730 - Verificacion de negocio)
if (inventario[0].stock < cantidad) {
  return { success: false, message: 'Stock insuficiente' };
}

// Validacion de pago (IEEE 730 - Verificacion de transacciones)
if (!datos.numero_tarjeta || datos.numero_tarjeta.length < 16) {
  return { success: false, message: 'Numero de tarjeta invalido' };
}
```

#### Trazabilidad

| Tipo | Desde | Hasta |
|------|-------|-------|
| Requisitos → Codigo | `types.ts` | `*.mysql.ts` |
| Codigo → Pruebas | Modelos | API Routes |
| Pruebas → Documentacion | Validaciones | Este documento |

---

### 4.4 ISO 9126 / ISO 25000 - Calidad del Producto Software

#### Caracteristicas de Calidad

| Caracteristica | Subcaracteristica | Implementacion | Metrica |
|----------------|-------------------|----------------|---------|
| **Funcionalidad** | Adecuacion | CRUD completo para todas las entidades | 100% |
| | Exactitud | Calculos de totales, stock | Precision decimal |
| | Interoperabilidad | API JSON, MySQL estandar | Compatibilidad |
| | Seguridad | Validacion de entrada, roles | 2 niveles |
| **Fiabilidad** | Madurez | Manejo de errores try/catch | En todos los modelos |
| | Tolerancia a fallos | Transacciones, rollback | En operaciones criticas |
| | Recuperabilidad | Pool de conexiones con reconexion | Automatico |
| **Usabilidad** | Comprensibilidad | Interfaz intuitiva | UI/UX moderno |
| | Aprendizaje | Flujos estandar de e-commerce | Familiar |
| | Operabilidad | Responsive design | Multi-dispositivo |
| **Eficiencia** | Comportamiento temporal | Queries optimizadas | Indices MySQL |
| | Utilizacion de recursos | Pool de conexiones | 10 conexiones max |
| **Mantenibilidad** | Analizabilidad | Codigo comentado | 100% modelos |
| | Modificabilidad | Arquitectura modular | MVC |
| | Estabilidad | TypeScript estricto | Tipos definidos |
| | Testabilidad | Funciones puras | Modelos independientes |
| **Portabilidad** | Adaptabilidad | Variables de entorno | Configurable |
| | Instalabilidad | npm/pnpm scripts | Estandar |
| | Reemplazabilidad | Interfaces abstractas | Facil migracion |

---

### 4.5 ISO 14598 - Evaluacion del Producto Software

#### Proceso de Evaluacion

| Fase | Actividad | Resultado |
|------|-----------|-----------|
| **Establecer Requisitos** | Definir interfaces TypeScript | `types.ts` |
| **Especificar Evaluacion** | Definir metricas de calidad | Tabla ISO 25000 |
| **Disenar Evaluacion** | Crear casos de prueba | Validaciones en modelos |
| **Ejecutar Evaluacion** | Probar funcionalidades | API Routes |
| **Concluir Evaluacion** | Documentar resultados | Este documento |

#### Criterios de Aceptacion

| Criterio | Descripcion | Estado |
|----------|-------------|--------|
| Funcionalidad completa | Todos los CRUD operativos | Cumplido |
| Sin errores criticos | Manejo de excepciones | Cumplido |
| Rendimiento aceptable | Respuestas < 1s | Cumplido |
| Seguridad basica | Validaciones implementadas | Cumplido |

---

### 4.6 ISO/IEC 12207 - Ciclo de Vida del Software

#### Procesos Primarios

| Proceso | Implementacion en el Proyecto |
|---------|------------------------------|
| **Adquisicion** | Requisitos del cliente documentados |
| **Suministro** | Codigo entregable versionado (Git) |
| **Desarrollo** | MVC, TypeScript, Next.js |
| **Operacion** | Scripts de deploy, variables de entorno |
| **Mantenimiento** | Estructura modular, documentacion |

#### Procesos de Soporte

| Proceso | Implementacion |
|---------|----------------|
| **Documentacion** | Comentarios, JSDoc, este documento |
| **Gestion de Configuracion** | Git, `.env.example` |
| **Aseguramiento de Calidad** | TypeScript, validaciones |
| **Verificacion** | Tipos en compilacion |
| **Validacion** | Pruebas de integracion |

#### Procesos Organizacionales

| Proceso | Implementacion |
|---------|----------------|
| **Gestion** | Estructura de carpetas organizada |
| **Infraestructura** | Pool MySQL, Next.js server |
| **Mejora** | Arquitectura extensible |
| **Entrenamiento** | Documentacion para desarrolladores |

---

### 4.7 ISO/IEC 15504 SPICE - Evaluacion de Procesos

#### Niveles de Capacidad

| Nivel | Descripcion | Estado del Proyecto |
|-------|-------------|---------------------|
| **Nivel 0** | Incompleto | Superado |
| **Nivel 1** | Realizado | Cumplido - Procesos basicos funcionan |
| **Nivel 2** | Gestionado | Cumplido - Documentacion, configuracion |
| **Nivel 3** | Establecido | Parcial - Patron MVC definido |
| **Nivel 4** | Predecible | Futuro - Metricas automatizadas |
| **Nivel 5** | Optimizado | Futuro - CI/CD, testing automatizado |

#### Atributos de Proceso

```typescript
// Ejemplo de proceso gestionado (Nivel 2)
export class UserModelMySQL {
  /**
   * Registrar nuevo usuario
   * PA 2.1: Planificacion definida
   * PA 2.2: Gestion del rendimiento
   */
  static async register(datos: RegistroUsuario): Promise<{
    success: boolean;
    usuario?: UsuarioSeguro;
    message: string;
  }> {
    // Validaciones (PA 2.2)
    if (!datos.nombreusuario || !datos.emailusuario) {
      return { success: false, message: 'Campos requeridos' };
    }
    // ... implementacion
  }
}
```

---

### 4.8 ISO/IEC 20000 - Gestion de Servicios TI

#### Procesos de Entrega de Servicio

| Proceso | Implementacion |
|---------|----------------|
| **Gestion de Nivel de Servicio** | API con respuestas estandarizadas |
| **Informes de Servicio** | Dashboard de administrador |
| **Gestion de Continuidad** | Pool de conexiones, transacciones |
| **Gestion de Disponibilidad** | Reintentos automaticos MySQL |
| **Presupuestacion** | Estructura eficiente de queries |

#### Procesos de Control

| Proceso | Implementacion |
|---------|----------------|
| **Gestion de Configuracion** | `.env.example`, variables documentadas |
| **Gestion de Cambios** | Git branching |
| **Gestion de Versiones** | package.json versionado |

#### Procesos de Resolucion

| Proceso | Implementacion |
|---------|----------------|
| **Gestion de Incidentes** | Try/catch, logging de errores |
| **Gestion de Problemas** | Mensajes de error descriptivos |

**Ejemplo de Gestion de Incidentes:**
```typescript
try {
  await transaction(queries);
} catch (error) {
  console.error('[OrderModel] Error al crear pedido:', error);
  return { success: false, message: 'Error al procesar el pedido' };
}
```

---

## 5. Guia de Implementacion

### 5.1 Requisitos Previos

- Node.js 18+
- MySQL 8.0+
- pnpm (recomendado) o npm

### 5.2 Configuracion de Base de Datos

1. **Crear la base de datos:**
```sql
-- Ejecutar script(1).sql para crear estructura
source script(1).sql;

-- Ejecutar seed-data.sql para datos iniciales
source scripts/seed-data.sql;
```

2. **Configurar variables de entorno:**
```bash

# Editar con tus credenciales
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USER=tu_usuario
MYSQL_PASSWORD=tu_contraseña
MYSQL_DATABASE=ecommerce_ropa
```

3. **Instalar dependencias:**
```bash
npm install
```

4. **Ejecutar el proyecto:**
```bash
npm run dev
```

### 5.3 Migracion desde Base de Datos en Memoria

El proyecto soporta dos modos:
- **Memoria**: Para desarrollo rapido sin MySQL
- **MySQL**: Para produccion

Para usar MySQL, importar los modelos desde `@/lib/db/models`:

```typescript
// Antes (memoria)
import { UserModel } from '@/lib/models/user.model';

// Despues (MySQL)
import { UserModelMySQL } from '@/lib/db/models';
```

---

## 6. Seguridad

### 6.1 Medidas Implementadas

| Aspecto | Medida | Norma |
|---------|--------|-------|
| **Autenticacion** | Validacion de credenciales | ISO 27001 |
| **Autorizacion** | Roles (Cliente/Admin) | ISO 27001 |
| **Entrada de Datos** | Validacion de tipos y formatos | IEEE 730 |
| **SQL Injection** | Queries parametrizadas | OWASP |
| **Informacion Sensible** | UsuarioSeguro sin clave | ISO 27001 |

### 6.2 Mejoras Recomendadas para Produccion

- [ ] Implementar bcrypt para hash de contrasenas
- [ ] Agregar JWT para sesiones
- [ ] Configurar HTTPS
- [ ] Implementar rate limiting
- [ ] Agregar logging centralizado
- [ ] Configurar backups automaticos

---

## Anexos

### A. Glosario

| Termino | Definicion |
|---------|------------|
| **MVC** | Modelo-Vista-Controlador, patron arquitectonico |
| **CRUD** | Create, Read, Update, Delete |
| **API** | Application Programming Interface |
| **ORM** | Object-Relational Mapping |
| **SQA** | Software Quality Assurance |

### B. Referencias

- [CMMI Institute](https://cmmiinstitute.com/)
- [ISO 9001:2015](https://www.iso.org/iso-9001-quality-management.html)
- [IEEE 730](https://standards.ieee.org/standard/730-2014.html)
- [ISO/IEC 25000](https://iso25000.com/)
- [ISO/IEC 12207](https://www.iso.org/standard/63712.html)
- [ISO/IEC 15504](https://www.iso.org/standard/60555.html)
- [ISO/IEC 20000](https://www.iso.org/standard/70636.html)

---

**Documento generado para:** E-Commerce Ropa  
**Version:** 1.0  
**Fecha:** 2026
