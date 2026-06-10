# 📚 BiblioTech - Gestión de Biblioteca ESCOM

Plataforma web enfocada en la comunidad universitaria (alumnos y profesores) del Instituto Politécnico Nacional, diseñada para modernizar y facilitar la administración del catálogo de libros. El sistema permite a los usuarios explorar millones de ejemplares en la red global mediante una API externa y gestionar solicitudes de préstamos físicos del acervo local.

**Asignatura:** Bases de Datos
**Grupo:** 3CV2
**Profesor:** Gabriel Hurtado Avilés

## 👨‍💻 Equipo de Desarrollo
* González Ortiz Iker Saúl
* Juárez Bobadilla Miguel Isaí

---

## 🚀 Enlaces del Proyecto
* **Código Fuente:** [Repositorio en GitHub](https://github.com/xsuik33/xsuik33.github.io)
* **Demo en Vivo:** [BiblioTech Web](https://xsuik33.github.io)

---

## 🛠️ Tecnologías Implementadas

* **Base de Datos / Backend:** PostgreSQL alojado en Supabase.
* **Autenticación:** Supabase Auth (Integración segura de sesiones).
* **Consumo de Datos Externos:** Open Library API REST.
* **Frontend:** HTML5, CSS3 avanzado (Flexbox/Grid, variables CSS) y JavaScript vanilla (Fetch API, manipulación dinámica del DOM).
* **Despliegue:** GitHub Pages.

---

## 📖 Evolución y Metodología del Proyecto

El desarrollo de BiblioTech se llevó a cabo siguiendo una metodología estructurada desde la conceptualización de los datos hasta la implementación de la interfaz de usuario.

### Fase 1: Análisis y Problemática
El proyecto nació para resolver tres problemas críticos identificados en el control manual de una biblioteca:
1. **Certeza sobre la posesión:** Necesidad de saber exactamente qué usuario tiene qué código de barras y la fecha esperada de devolución.
2. **Pérdida de credenciales:** Transición a un registro digital utilizando la CURP o número de boleta/empleado.
3. **Cálculo de multas:** Automatización de las penalizaciones por retraso.

### Fase 2: Modelo Entidad-Relación Extendido (EER)
El modelo básico evolucionó para soportar dependencias de existencia y jerarquías del mundo real mediante el Modelo EER.

![Diagrama EER - Notación Peter Chen](docs/images/diagrama-eer-chen.jpg)
*Figura 1: Diagrama EER en notación Peter Chen resaltando la semántica del negocio.*

![Diagrama EER - Notación Crow's Foot](docs/images/diagrama-eer-crows.jpg)
*Figura 2: Diagrama EER en notación Crow's Foot, orientado a la implementación técnica.*

#### 📐 Explicación Detallada del Diseño Conceptual
Para garantizar la integridad y evitar redundancias, se aplicaron los siguientes conceptos avanzados:

* **Entidades Débiles:** * `EJEMPLAR`: No posee autonomía semántica; su existencia depende directamente de un `LIBRO` (entidad fuerte). Su identificador (No. Copia) es un discriminante parcial que solo cobra sentido al combinarse con el ISBN del libro.
  * `MULTA`: Se clasifica como débil por su naturaleza transaccional. No puede existir una sanción económica en el sistema si no hay un registro de Préstamo previo que la sustente.
* **Jerarquía de Especialización (Herencia):**
  * Se implementó una jerarquía de Especialización **Total y Disjunta** para la entidad `USUARIO`. 
  * Es *Disjunta* porque un registro no puede ser simultáneamente Alumno y Profesor, evitando conflictos en los privilegios de préstamo. 
  * Es *Total* porque no se permiten usuarios "genéricos"; todo usuario debe tener un rol definido para aplicar las reglas de negocio.
* **Cardinalidades Críticas:**
  * [cite_start]`Libro - Ejemplar (1, N)`: Un libro debe tener al menos un soporte físico para ser registrado, permitiendo el crecimiento del inventario[cite: 2783].
  * [cite_start]`Ejemplar - Préstamo (0, 1)`: Garantiza matemáticamente que un objeto físico específico solo pueda estar asociado a un préstamo activo a la vez, evitando duplicidades de entrega[cite: 2785].
* [cite_start]**Relaciones de Orden Superior (Agregación):** La relación ternaria `ASIGNA` vincula al Bibliotecario, el Ejemplar y la Estantería para crear un registro único de Alta de Inventario y permitir auditorías precisas[cite: 2654, 2835].

### Fase 3: Modelo Relacional y Restricciones (DDL)
El modelo conceptual se tradujo a esquemas relacionales exactos en PostgreSQL, aplicando reglas matemáticas estrictas para la consistencia de los datos.

![Esquema del Modelo Relacional](docs/images/diagrama-relacional.png)
*Figura 3: Diagrama del Esquema Relacional consolidado.*

#### ⚙️ Explicación Detallada de la Transformación Relacional
* [cite_start]**Transformación de Entidades Fuertes:** Entidades como `USUARIO`, `LIBRO` y `BIBLIOTECARIO` se transformaron en tablas independientes conservando sus llaves primarias originales[cite: 3286].
* [cite_start]**Transformación de Entidades Débiles (Borrado en Cascada):** Las entidades `EJEMPLAR` y `MULTA` se implementaron manteniendo su dependencia estructural mediante llaves foráneas y la restricción `ON DELETE CASCADE`[cite: 3287, 3302]. [cite_start]Si un libro es retirado del catálogo, sus copias físicas se eliminan automáticamente[cite: 3093].
* [cite_start]**Resolución de Herencia:** Los subtipos `ALUMNO` y `PROFESOR` se crearon como tablas independientes[cite: 3288]. [cite_start]Ambas utilizan la `CURP` simultáneamente como Llave Primaria (PK) y Llave Foránea (FK) apuntando al supertipo `USUARIO`, lo que permite conservar atributos específicos (Boleta o No. Empleado) sin generar valores nulos (`NULL`) masivos en una tabla general[cite: 3165, 3175, 3101, 3102].
* [cite_start]**Resolución de Relaciones N:M:** La autoría de los libros se resolvió creando la tabla asociativa `ESCRITO_POR` para conectar libros y autores sin redundancia de datos[cite: 3292, 3040].
* **Restricciones de Dominio (Constraints):**
  * [cite_start]`CHECK`: Se implementaron validaciones lógicas, como asegurar que la fecha de devolución no sea menor a la de salida [cite: 3473][cite_start], que el formato de correo contenga una '@' [cite: 3462][cite_start], y que el estado físico de los libros solo sea 'Excelente', 'Desgastado' o 'Dañado'[cite: 3485].
  * [cite_start]`UNIQUE`: Aplicado a correos electrónicos y números de boleta/empleado para evitar duplicidad de cuentas en el sistema[cite: 3435].
  * [cite_start]`DEFAULT`: Asignación automática de fechas actuales en préstamos y estados iniciales ('Pendiente' en multas) para agilizar la operación[cite: 3409, 3416].

### Fase 4: Desarrollo de la Interfaz Web
Finalmente, esta robusta base de datos se conectó con una interfaz de usuario interactiva para el usuario final.

**✨ Funcionalidades Destacadas del Frontend:**
* Registro e inicio de sesión seguro validado para la comunidad universitaria.
* Interfaz con soporte nativo para **Modo Claro / Modo Oscuro**.
* Paginación dinámica y renderizado de tarjetas sin recargar la página.
* Sistema de internacionalización (i18n) para Español, Inglés y Francés.
* Préstamos automatizados con cálculo de fecha de devolución a 7 días.

---

## 💻 Galería del Sistema

<details>
<summary>🖼️ Ver capturas de pantalla de la plataforma</summary>

| Pantalla de Inicio | Inicio de Sesión |
|:---:|:---:|
| <img loading="lazy" src="https://github.com/xsuik33/xsuik33.github.io/blob/main/Page.png" alt="Vista principal" width="400"/> | <img loading="lazy" src="https://github.com/xsuik33/xsuik33.github.io/blob/main/Login.png" alt="Login" width="400"/> |

| Registro de Usuario | Vista de Catálogo / Sección |
|:---:|:---:|
| <img loading="lazy" src="https://github.com/xsuik33/xsuik33.github.io/blob/main/Register.png" alt="Registro" width="400"/> | <img loading="lazy" src="https://github.com/xsuik33/xsuik33.github.io/blob/main/Section.png" alt="Vista de Sección" width="400"/> |

| Vista Previa del Libro |
|:---:|
| <img loading="lazy" src="https://github.com/xsuik33/xsuik33.github.io/blob/main/Preview.png" alt="Detalle del Libro" width="400"/> |

</details>
