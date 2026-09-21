# Examen: Base de datos SST/PESV en PostgreSQL

Este documento recoge el estado completo del examen: la base de datos (tablas y datos de prueba), y las respuestas a cada bloque de consultas, indicando cuáles ya están resueltas y cuáles faltan.

**Conexión:**
```
psql -h postgres_db -p 5432 -U bkseducate -d trabajo
```

---

## Estado general

| Sección | Estado |
|---|---|
| Tablas (modelo físico) | ✅ Completado |
| Datos de prueba | ✅ Completado |
| 1. Consultas SQL básicas (15) | ✅ Completado |
| 2. Consultas SQL intermedias (20) | ✅ Completado |
| 3. Consultas SQL avanzadas (25) | ✅ Completado |
| 4. Consultas de vistas y vistas materializadas (8) | ✅ Completado |
| 5. Procedimientos almacenados (15) | ✅ Completado |
| 6. Funciones almacenadas (8) | ✅ Completado |
| 7. Triggers (15) | ✅ Completado |

---

## Tablas (modelo físico)

```sql
CREATE TABLE countries (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE departments (
    id          SERIAL PRIMARY KEY,
    country_id  INTEGER NOT NULL REFERENCES countries(id),
    name        VARCHAR(100) NOT NULL,
    UNIQUE (country_id, name)
);

CREATE TABLE cities (
    id              SERIAL PRIMARY KEY,
    department_id   INTEGER NOT NULL REFERENCES departments(id),
    name            VARCHAR(100) NOT NULL,
    UNIQUE (department_id, name)
);

CREATE TABLE tenant_sizes (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE type_system_sst (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(50) NOT NULL UNIQUE,
    description VARCHAR(200)
);

CREATE TABLE phva_stages (
    id             SERIAL PRIMARY KEY,
    name           VARCHAR(20) NOT NULL UNIQUE,
    display_order  SMALLINT NOT NULL UNIQUE
);

CREATE TABLE tenants (
    id              SERIAL PRIMARY KEY,
    name            VARCHAR(150) NOT NULL,
    nit             VARCHAR(20) NOT NULL UNIQUE,
    contact_email   VARCHAR(150),
    contact_phone   VARCHAR(30),
    tenant_size_id  INTEGER REFERENCES tenant_sizes(id),
    city_id         INTEGER REFERENCES cities(id),
    status          BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE positions (
    id          SERIAL PRIMARY KEY,
    tenant_id   INTEGER NOT NULL REFERENCES tenants(id),
    description VARCHAR(100) NOT NULL,
    UNIQUE (tenant_id, description)
);

CREATE TABLE persons (
    id          SERIAL PRIMARY KEY,
    tenant_id   INTEGER NOT NULL REFERENCES tenants(id),
    position_id INTEGER REFERENCES positions(id),
    first_name  VARCHAR(80) NOT NULL,
    last_name   VARCHAR(80) NOT NULL,
    email       VARCHAR(150) NOT NULL UNIQUE,
    status      BOOLEAN NOT NULL DEFAULT TRUE,
    created_at  TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE tenantsystems (
    id          SERIAL PRIMARY KEY,
    tenant_id   INTEGER NOT NULL REFERENCES tenants(id),
    system_id   INTEGER NOT NULL REFERENCES type_system_sst(id),
    enabled_at  TIMESTAMP NOT NULL DEFAULT NOW(),
    UNIQUE (tenant_id, system_id)
);

CREATE TABLE modules (
    id              SERIAL PRIMARY KEY,
    system_id       INTEGER NOT NULL REFERENCES type_system_sst(id),
    title           VARCHAR(150) NOT NULL,
    description     VARCHAR(300),
    display_order   SMALLINT NOT NULL DEFAULT 0,
    UNIQUE (system_id, title)
);

CREATE TABLE tenant_modules (
    id          SERIAL PRIMARY KEY,
    tenant_id   INTEGER NOT NULL REFERENCES tenants(id),
    module_id   INTEGER NOT NULL REFERENCES modules(id),
    assigned_at TIMESTAMP NOT NULL DEFAULT NOW(),
    UNIQUE (tenant_id, module_id)
);

CREATE TABLE formats_sst (
    id          SERIAL PRIMARY KEY,
    module_id   INTEGER NOT NULL REFERENCES modules(id),
    name        VARCHAR(150) NOT NULL,
    description VARCHAR(300)
);

CREATE TABLE templates (
    id              SERIAL PRIMARY KEY,
    system_id       INTEGER NOT NULL REFERENCES type_system_sst(id),
    module_id       INTEGER NOT NULL REFERENCES modules(id),
    phva_stage_id   INTEGER NOT NULL REFERENCES phva_stages(id),
    format_id       INTEGER REFERENCES formats_sst(id),
    name            VARCHAR(150) NOT NULL,
    description     VARCHAR(300)
);

CREATE TABLE tenanttemplates (
    id              SERIAL PRIMARY KEY,
    tenant_id       INTEGER NOT NULL REFERENCES tenants(id),
    template_id     INTEGER NOT NULL REFERENCES templates(id),
    system_id       INTEGER NOT NULL REFERENCES type_system_sst(id),
    phva_stage_id   INTEGER NOT NULL REFERENCES phva_stages(id),
    format_id       INTEGER REFERENCES formats_sst(id),
    status          VARCHAR(20) NOT NULL DEFAULT 'no_iniciado'
                    CHECK (status IN ('no_iniciado','borrador','finalizado','pendiente')),
    assigned_at     TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_by      INTEGER REFERENCES persons(id)
);

CREATE TABLE evaluations (
    id          SERIAL PRIMARY KEY,
    tenant_id   INTEGER NOT NULL REFERENCES tenants(id),
    module_id   INTEGER NOT NULL REFERENCES modules(id),
    name        VARCHAR(150) NOT NULL,
    description VARCHAR(300),
    created_at  TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE editing_locks (
    id                  SERIAL PRIMARY KEY,
    tenanttemplate_id   INTEGER NOT NULL REFERENCES tenanttemplates(id),
    locked_by           INTEGER NOT NULL REFERENCES persons(id),
    locked_at           TIMESTAMP NOT NULL DEFAULT NOW(),
    expires_at          TIMESTAMP NOT NULL,
    active              BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE TABLE tenant_audit (
    id          SERIAL PRIMARY KEY,
    tenant_id   INTEGER NOT NULL REFERENCES tenants(id),
    field_name  VARCHAR(60),
    old_value   TEXT,
    new_value   TEXT,
    operation   VARCHAR(20),
    changed_by  VARCHAR(80),
    changed_at  TIMESTAMP NOT NULL DEFAULT NOW()
);
```

---

## Datos de prueba

```sql
INSERT INTO countries (name) VALUES ('Colombia');

INSERT INTO departments (country_id, name) VALUES
(1, 'Antioquia'), (1, 'Cundinamarca'), (1, 'Valle del Cauca'), (1, 'Atlántico');

INSERT INTO cities (department_id, name) VALUES
(1, 'Medellín'), (1, 'Envigado'),
(2, 'Bogotá'), (2, 'Chía'),
(3, 'Cali'), (3, 'Palmira'),
(4, 'Barranquilla');

INSERT INTO tenant_sizes (name) VALUES ('Microempresa'), ('Pequeña'), ('Mediana'), ('Grande');

INSERT INTO type_system_sst (name, description) VALUES
('SG-SST', 'Sistema de Gestión de Seguridad y Salud en el Trabajo'),
('PESV', 'Plan Estratégico de Seguridad Vial');

INSERT INTO phva_stages (name, display_order) VALUES
('Planear', 1), ('Hacer', 2), ('Verificar', 3), ('Actuar', 4);

INSERT INTO tenants (name, nit, contact_email, contact_phone, tenant_size_id, city_id, status, created_at) VALUES
('Constructora Andes S.A.S',      '900111222-1', 'contacto@andes.com',      '3001112233', 3, 1, TRUE,  '2024-01-15'),
('Transportes del Valle Ltda',    '900333444-5', 'contacto@transvalle.com', '3002223344', 2, 5, TRUE,  '2024-03-10'),
('Logística Caribe S.A.S',        '900555666-7', 'contacto@logcaribe.com',  '3003334455', 4, 7, TRUE,  '2024-05-20'),
('Textiles Bogotá S.A.S',         '900777888-9', 'contacto@textbog.com',    '3004445566', 1, 3, FALSE, '2023-11-02'),
('Servicios Industriales Chía',   '900999000-2', 'contacto@servchia.com',   '3005556677', 2, 4, TRUE,  '2024-07-08'),
('Papelería El Dorado S.A.S',     '900222111-3', 'contacto@eldorado.com',   '3006667788', 1, 1, TRUE,  '2024-08-01');

INSERT INTO positions (tenant_id, description) VALUES
(1,'Gerente General'), (1,'Coordinador SST'), (1,'Operario'),
(2,'Gerente General'), (2,'Conductor'), (2,'Coordinador PESV'),
(3,'Gerente General'), (3,'Auxiliar Logístico'),
(4,'Gerente General'), (4,'Coordinador SST'),
(5,'Gerente General'), (5,'Coordinador SST'), (5,'Coordinador PESV');

INSERT INTO persons (tenant_id, position_id, first_name, last_name, email, status, created_at) VALUES
(1, 1, 'Carlos', 'Ramírez', 'carlos.ramirez@andes.com', TRUE, '2024-01-16'),
(1, 2, 'Ana',    'Gómez',   'ana.gomez@andes.com',      TRUE, '2024-01-20'),
(1, 3, 'Luis',   'Torres',  'luis.torres@andes.com',    TRUE, '2024-02-01'),
(1, 3, 'Sofía',  'Mejía',   'sofia.mejia@andes.com',    TRUE, '2024-02-10'),
(1, 3, 'Andrés', 'Vélez',   'andres.velez@andes.com',   TRUE, '2024-02-12'),
(2, 4, 'María',  'Pérez',   'maria.perez@transvalle.com', TRUE, '2024-03-11'),
(2, 5, 'Jorge',  'Díaz',    'jorge.diaz@transvalle.com',  TRUE, '2024-03-15'),
(2, 6, 'Paula',  'Suárez',  'paula.suarez@transvalle.com',TRUE, '2024-04-01'),
(3, 7, 'Andrés', 'López',   'andres.lopez@logcaribe.com', TRUE, '2024-05-21'),
(3, 8, 'Diana',  'Castro',  'diana.castro@logcaribe.com', FALSE,'2024-06-01'),
(5, 11,'Camila', 'Rojas',   'camila.rojas@servchia.com',  TRUE, '2024-07-09'),
(5, 12,'Felipe', 'Mora',    'felipe.mora@servchia.com',   TRUE, '2024-07-10');

INSERT INTO tenantsystems (tenant_id, system_id) VALUES
(1,1), (2,1), (2,2), (3,1), (5,1), (5,2), (6,1);

INSERT INTO modules (system_id, title, description, display_order) VALUES
(1, 'Política SST', 'Definición de política de seguridad y salud', 1),
(1, 'Identificación de peligros', 'Matriz IPERC', 2),
(1, 'Capacitaciones', 'Programa anual de capacitación', 3),
(2, 'Diagnóstico vial', 'Diagnóstico inicial PESV', 1),
(2, 'Plan de acción vial', 'Plan estratégico de seguridad vial', 2),
(1, 'Auditorías internas', 'Módulo sin organizaciones asignadas', 4);

INSERT INTO tenant_modules (tenant_id, module_id) VALUES
(1,1), (1,2), (1,3),
(2,1), (2,4), (2,5),
(3,1),
(5,1), (5,4),
(6,1);

INSERT INTO formats_sst (module_id, name, description) VALUES
(1, 'Formato Política SST', 'Documento formal de política'),
(2, 'Matriz de riesgos', 'Formato IPERC'),
(3, 'Registro de capacitación', 'Control de asistencia'),
(4, 'Diagnóstico vial inicial', 'Formato de diagnóstico'),
(5, 'Plan de acción PESV', 'Formato de plan estratégico');

INSERT INTO templates (system_id, module_id, phva_stage_id, format_id, name, description) VALUES
(1,1,1,1,'Plantilla Política SST', 'Planear'),
(1,2,1,2,'Plantilla Matriz IPERC', 'Planear'),
(1,3,2,3,'Plantilla Capacitación', 'Hacer'),
(1,1,3,1,'Plantilla Revisión Política', 'Verificar'),
(1,2,4,2,'Plantilla Actualización IPERC', 'Actuar'),
(2,4,1,4,'Plantilla Diagnóstico Vial', 'Planear'),
(2,5,2,5,'Plantilla Plan de Acción PESV', 'Hacer'),
(2,5,3,5,'Plantilla Seguimiento PESV', 'Verificar');

INSERT INTO tenanttemplates (tenant_id, template_id, system_id, phva_stage_id, format_id, status, assigned_at, updated_at) VALUES
(1, 1, 1, 1, 1, 'finalizado',  '2024-02-01', '2024-02-10'),
(1, 2, 1, 1, 2, 'finalizado',  '2024-02-01', '2024-02-15'),
(1, 3, 1, 2, 3, 'borrador',    '2024-02-05', '2024-03-01'),
(1, 4, 1, 3, 1, 'no_iniciado', '2024-02-05', '2024-02-05'),
(1, 5, 1, 4, 2, 'finalizado',  '2024-03-05', '2024-03-15'),
(2, 1, 1, 1, 1, 'finalizado',  '2024-03-12', '2024-03-20'),
(2, 2, 1, 1, 2, 'pendiente',   '2024-04-10', '2024-04-10'),
(2, 6, 2, 1, 4, 'finalizado',  '2024-03-16', '2024-03-25'),
(2, 7, 2, 2, 5, 'pendiente',   '2024-04-02', '2024-04-02'),
(2, 8, 2, 3, 5, 'no_iniciado', '2024-04-02', '2024-04-02'),
(3, 1, 1, 1, 1, 'borrador',    '2024-05-22', '2024-06-01'),
(3, 4, 1, 3, 1, 'pendiente',   '2024-06-05', '2024-06-05'),
(5, 1, 1, 1, 1, 'finalizado',  '2024-07-11', '2024-07-20'),
(5, 6, 2, 1, 4, 'finalizado',  '2024-07-11', '2024-07-22'),
(5, 7, 2, 2, 5, 'borrador',    '2024-07-15', '2024-08-01');

INSERT INTO evaluations (tenant_id, module_id, name, description) VALUES
(1, 3, 'Evaluación de capacitación inicial', 'Instrumento de evaluación de conocimientos'),
(2, 5, 'Evaluación plan de acción PESV', 'Evaluación del plan estratégico vial');

INSERT INTO editing_locks (tenanttemplate_id, locked_by, locked_at, expires_at, active) VALUES
(3, 2, NOW(), NOW() + INTERVAL '30 minutes', TRUE);
```

**Nota:** en el desarrollo también se insertaron más países de referencia (México, Argentina, Chile, Perú, Ecuador, Venezuela, Brasil, Estados Unidos, España, Panamá) y un documento adicional de prueba para Constructora Andes al validar el `REFRESH MATERIALIZED VIEW`, opcional para el examen.

---

## 1. Consultas SQL básicas ✅

**1. Todos los registros de `tenants`**
```sql
SELECT * FROM tenants;
```

**2. Nombre, correo y teléfono de las organizaciones**
```sql
SELECT name, contact_email, contact_phone FROM tenants;
```

**3. Nombres, apellidos y correo de las personas**
```sql
SELECT first_name, last_name, email FROM persons;
```

**4. Personas activas**
```sql
SELECT * FROM persons WHERE status = TRUE;
```

**5. Organizaciones cuyo nombre contenga una palabra**
```sql
SELECT * FROM tenants WHERE name ILIKE '%Andes%';
```

**6. Países ordenados alfabéticamente**
```sql
SELECT * FROM countries ORDER BY name ASC;
```

**7. Departamentos de un país determinado**
```sql
SELECT * FROM departments WHERE country_id = 1;
```

**8. Ciudades de un departamento específico**
```sql
SELECT * FROM cities WHERE department_id = 1;
```

**9. Cargos ordenados por descripción**
```sql
SELECT * FROM positions ORDER BY description ASC;
```

**10. Personas de una organización según `tenant_id`**
```sql
SELECT * FROM persons WHERE tenant_id = 1;
```

**11. Organizaciones activas**
```sql
SELECT * FROM tenants WHERE status = TRUE;
```

**12. Organizaciones registradas en un rango de fechas**
```sql
SELECT * FROM tenants WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';
```

**13. Tamaños de empresa**
```sql
SELECT * FROM tenant_sizes;
```

**14. Tipos de sistemas SST**
```sql
SELECT * FROM type_system_sst;
```

**15. Módulos con título, descripción y orden**
```sql
SELECT title, description, display_order FROM modules ORDER BY display_order;
```

---

## 2. Consultas SQL intermedias ✅

**1. Personas con el nombre de su organización**
```sql
SELECT personas.first_name || ' ' || personas.last_name AS nombre_completo, organizaciones.name AS nombre_organizacion
FROM persons AS personas
JOIN tenants AS organizaciones ON organizaciones.id = personas.tenant_id;
```

**2. Personas con su cargo**
```sql
SELECT personas.first_name || ' ' || personas.last_name AS nombre_completo, cargos.description AS cargo
FROM persons AS personas
JOIN positions AS cargos ON cargos.id = personas.position_id;
```

**3. Organizaciones con su tamaño de empresa**
```sql
SELECT organizaciones.name AS nombre_organizacion, tamanos.name AS tamano_empresa
FROM tenants AS organizaciones
JOIN tenant_sizes AS tamanos ON tamanos.id = organizaciones.tenant_size_id;
```

**4. Organizaciones con ciudad, departamento y país**
```sql
SELECT organizaciones.name AS nombre_organizacion, ciudades.name AS ciudad, departamentos.name AS departamento, paises.name AS pais
FROM tenants AS organizaciones
JOIN cities AS ciudades ON ciudades.id = organizaciones.city_id
JOIN departments AS departamentos ON departamentos.id = ciudades.department_id
JOIN countries AS paises ON paises.id = departamentos.country_id;
```

**5. Total de personas por organización**
```sql
SELECT organizaciones.name AS nombre_organizacion, COUNT(personas.id) AS total_personas
FROM tenants AS organizaciones
LEFT JOIN persons AS personas ON personas.tenant_id = organizaciones.id
GROUP BY organizaciones.name;
```

**6. Organizaciones con más de 2 personas**
```sql
SELECT organizaciones.name AS nombre_organizacion, COUNT(personas.id) AS total_personas
FROM tenants AS organizaciones
JOIN persons AS personas ON personas.tenant_id = organizaciones.id
GROUP BY organizaciones.name
HAVING COUNT(personas.id) > 2;
```

**7. Organización y módulo asignado**
```sql
SELECT organizaciones.name AS nombre_organizacion, modulos.title AS nombre_modulo
FROM tenant_modules AS modulos_organizacion
JOIN tenants AS organizaciones ON organizaciones.id = modulos_organizacion.tenant_id
JOIN modules AS modulos ON modulos.id = modulos_organizacion.module_id;
```

**8. Total de módulos por organización**
```sql
SELECT organizaciones.name AS nombre_organizacion, COUNT(modulos_organizacion.module_id) AS total_modulos
FROM tenants AS organizaciones
JOIN tenant_modules AS modulos_organizacion ON modulos_organizacion.tenant_id = organizaciones.id
GROUP BY organizaciones.name;
```

**9. Sistemas SST habilitados por organización**
```sql
SELECT organizacion.name AS nombre_organizacion, sst.name AS sistema_sst
FROM tenantsystems AS sistema_organizacion
JOIN tenants AS organizacion ON organizacion.id = sistema_organizacion.tenant_id
JOIN type_system_sst AS sst ON sst.id = sistema_organizacion.system_id;
```

**10. Módulos junto con el sistema SST al que pertenecen**
```sql
SELECT modulo.title AS nombre_modulo, sst.name AS sistema_sst
FROM modules AS modulo
JOIN type_system_sst AS sst ON sst.id = modulo.system_id;
```

**11. Formatos con su módulo**
```sql
SELECT formatos.name AS nombre_formato, modulos.title AS nombre_modulo
FROM formats_sst AS formatos
JOIN modules AS modulos ON modulos.id = formatos.module_id;
```

**12. Total de formatos por módulo**
```sql
SELECT modulos.title AS nombre_modulo, COUNT(formatos.id) AS total_formatos
FROM modules AS modulos
JOIN formats_sst AS formatos ON formatos.module_id = modulos.id
GROUP BY modulos.title;
```

**13. Plantillas asignadas con su organización**
```sql
SELECT organizaciones.name AS nombre_organizacion, plantillas.name AS nombre_plantilla
FROM tenanttemplates AS plantillas_organizacion
JOIN tenants AS organizaciones ON organizaciones.id = plantillas_organizacion.tenant_id
JOIN templates AS plantillas ON plantillas.id = plantillas_organizacion.template_id;
```

**14. Plantilla, organización, sistema SST y etapa PHVA**
```sql
SELECT plantilla.name AS nombre_plantilla, organizacion.name AS nombre_organizacion, sst.name AS sistema_sst, phva.name AS etapa_phva
FROM tenanttemplates AS plantilla_organizacion
JOIN tenants AS organizacion ON organizacion.id = plantilla_organizacion.tenant_id
JOIN templates AS plantilla ON plantilla.id = plantilla_organizacion.template_id
JOIN type_system_sst AS sst ON sst.id = plantilla_organizacion.system_id
JOIN phva_stages AS phva ON phva.id = plantilla_organizacion.phva_stage_id;
```

**15. Total de plantillas por organización**
```sql
SELECT organizaciones.name AS nombre_organizacion, COUNT(plantillas_organizacion.id) AS total_plantillas
FROM tenants AS organizaciones
JOIN tenanttemplates AS plantillas_organizacion ON plantillas_organizacion.tenant_id = organizaciones.id
GROUP BY organizaciones.name;
```

**16. Organizaciones sin personas registradas**
```sql
SELECT organizacion.name AS nombre_organizacion
FROM tenants AS organizacion
LEFT JOIN persons AS persona ON persona.tenant_id = organizacion.id
WHERE persona.id IS NULL;
```

**17. Módulos no asignados a ninguna organización**
```sql
SELECT modulo.title AS nombre_modulo
FROM modules AS modulo
LEFT JOIN tenant_modules AS modulos_organizacion ON modulos_organizacion.module_id = modulo.id
WHERE modulos_organizacion.id IS NULL;
```

**18. Total de plantillas por etapa PHVA**
```sql
SELECT phva.name AS etapas_phva, COUNT(plantilla.id) AS plantillas_totales
FROM phva_stages AS phva
LEFT JOIN templates AS plantilla ON plantilla.phva_stage_id = phva.id
GROUP BY phva.name;
```

**19. Organizaciones registradas por ciudad**
```sql
SELECT ciudad.name AS ciudad, COUNT(organizacion.id) AS organizaciones_totales
FROM cities AS ciudad
LEFT JOIN tenants AS organizacion ON organizacion.city_id = ciudad.id
GROUP BY ciudad.name;
```

**20. Total de personas por organización y cargo**
```sql
SELECT organizaciones.name AS nombre_organizacion, cargos.description AS cargo, COUNT(personas.id) AS total_personas
FROM tenants AS organizaciones
JOIN persons AS personas ON personas.tenant_id = organizaciones.id
JOIN positions AS cargos ON cargos.id = personas.position_id
GROUP BY organizaciones.name, cargos.description;
```

---

## 3. Consultas SQL avanzadas ✅

**1. Organización con más personas registradas**
```sql
SELECT organizacion.name AS nombre_organizacion, COUNT(persona.id) AS total_personas
FROM tenants AS organizacion
JOIN persons AS persona ON persona.tenant_id = organizacion.id
GROUP BY organizacion.name
ORDER BY total_personas DESC
LIMIT 1;
```

**2. Organizaciones con personas por encima del promedio general**
```sql
SELECT organizacion.name AS nombre_organizacion, COUNT(persona.id) AS total_personas
FROM tenants AS organizacion
JOIN persons AS persona ON persona.tenant_id = organizacion.id
GROUP BY organizacion.name
HAVING COUNT(persona.id) > (
    SELECT AVG(total_por_organizacion.total_personas)
    FROM (
        SELECT COUNT(persona_interna.id) AS total_personas
        FROM tenants AS organizacion_interna
        LEFT JOIN persons AS persona_interna ON persona_interna.tenant_id = organizacion_interna.id
        GROUP BY organizacion_interna.id
    ) AS total_por_organizacion
);
```

**3. Organizaciones con todos los módulos de un sistema SST determinado**
```sql
SELECT organizacion.name AS nombre_organizacion
FROM tenants AS organizacion
JOIN tenant_modules AS modulo_organizacion ON modulo_organizacion.tenant_id = organizacion.id
JOIN modules AS modulo ON modulo.id = modulo_organizacion.module_id
JOIN type_system_sst AS sistema_sst ON sistema_sst.id = modulo.system_id
WHERE sistema_sst.name = 'PESV'
GROUP BY organizacion.id, organizacion.name
HAVING COUNT(DISTINCT modulo.id) = (
    SELECT COUNT(*)
    FROM modules
    JOIN type_system_sst ON type_system_sst.id = modules.system_id
    WHERE type_system_sst.name = 'PESV'
);
```

**4. Organizaciones con al menos un módulo pero sin plantillas asignadas**
```sql
SELECT DISTINCT organizacion.name AS nombre_organizacion
FROM tenants AS organizacion
JOIN tenant_modules AS modulo_organizacion ON modulo_organizacion.tenant_id = organizacion.id
LEFT JOIN tenanttemplates AS plantilla_organizacion ON plantilla_organizacion.tenant_id = organizacion.id
WHERE plantilla_organizacion.id IS NULL;
```

**5. Organizaciones con plantillas en todas las etapas PHVA**
```sql
SELECT organizacion.name AS nombre_organizacion
FROM tenants AS organizacion
JOIN tenanttemplates AS plantilla_organizacion ON plantilla_organizacion.tenant_id = organizacion.id
GROUP BY organizacion.id, organizacion.name
HAVING COUNT(DISTINCT plantilla_organizacion.phva_stage_id) = (SELECT COUNT(*) FROM phva_stages);
```

**6. Plantillas asignadas por organización discriminadas por etapa PHVA**
```sql
SELECT organizacion.name AS nombre_organizacion, etapa_phva.name AS etapa_phva, COUNT(plantilla_organizacion.id) AS total_plantillas
FROM tenanttemplates AS plantilla_organizacion
JOIN tenants AS organizacion ON organizacion.id = plantilla_organizacion.tenant_id
JOIN phva_stages AS etapa_phva ON etapa_phva.id = plantilla_organizacion.phva_stage_id
GROUP BY organizacion.name, etapa_phva.name;
```

**7. Cantidad de plantillas por etapa en columnas independientes**
```sql
SELECT organizacion.name AS nombre_organizacion,
       COUNT(*) FILTER (WHERE etapa_phva.name = 'Planear') AS planear,
       COUNT(*) FILTER (WHERE etapa_phva.name = 'Hacer') AS hacer,
       COUNT(*) FILTER (WHERE etapa_phva.name = 'Verificar') AS verificar,
       COUNT(*) FILTER (WHERE etapa_phva.name = 'Actuar') AS actuar
FROM tenanttemplates AS plantilla_organizacion
JOIN tenants AS organizacion ON organizacion.id = plantilla_organizacion.tenant_id
JOIN phva_stages AS etapa_phva ON etapa_phva.id = plantilla_organizacion.phva_stage_id
GROUP BY organizacion.name;
```

**8. Porcentaje que representa cada etapa PHVA sobre el total de plantillas**
```sql
SELECT organizacion.name AS nombre_organizacion, etapa_phva.name AS etapa_phva,
       COUNT(plantilla_organizacion.id) AS total_plantillas,
       ROUND(100.0 * COUNT(plantilla_organizacion.id) / SUM(COUNT(plantilla_organizacion.id)) OVER (PARTITION BY organizacion.id), 2) AS porcentaje_etapa
FROM tenanttemplates AS plantilla_organizacion
JOIN tenants AS organizacion ON organizacion.id = plantilla_organizacion.tenant_id
JOIN phva_stages AS etapa_phva ON etapa_phva.id = plantilla_organizacion.phva_stage_id
GROUP BY organizacion.id, organizacion.name, etapa_phva.name;
```

**9. Etapa PHVA con mayor cantidad de plantillas por organización**
```sql
SELECT nombre_organizacion, etapa_phva, total_plantillas
FROM (
    SELECT organizacion.name AS nombre_organizacion, etapa_phva.name AS etapa_phva,
           COUNT(plantilla_organizacion.id) AS total_plantillas,
           ROW_NUMBER() OVER (PARTITION BY organizacion.id ORDER BY COUNT(plantilla_organizacion.id) DESC) AS posicion
    FROM tenanttemplates AS plantilla_organizacion
    JOIN tenants AS organizacion ON organizacion.id = plantilla_organizacion.tenant_id
    JOIN phva_stages AS etapa_phva ON etapa_phva.id = plantilla_organizacion.phva_stage_id
    GROUP BY organizacion.id, organizacion.name, etapa_phva.name
) AS etapas_por_organizacion
WHERE posicion = 1;
```

**10. Porcentaje de documentos finalizados frente al total (vista de resumen)**
```sql
SELECT nombre_organizacion, total_documentos, finalizados,
       ROUND(100.0 * finalizados / NULLIF(total_documentos, 0), 2) AS porcentaje_cumplimiento
FROM vm_tenant_docs_summary;
```

**11. Organizaciones con cumplimiento por debajo del promedio general**
```sql
SELECT nombre_organizacion, porcentaje_cumplimiento
FROM vm_tenant_docs_summary
WHERE porcentaje_cumplimiento < (SELECT AVG(porcentaje_cumplimiento) FROM vm_tenant_docs_summary);
```

**12. Clasificación del cumplimiento en bajo, medio y alto**
```sql
SELECT nombre_organizacion, porcentaje_cumplimiento,
       CASE
           WHEN porcentaje_cumplimiento < 40 THEN 'Bajo'
           WHEN porcentaje_cumplimiento < 70 THEN 'Medio'
           ELSE 'Alto'
       END AS nivel_cumplimiento
FROM vm_tenant_docs_summary;
```

**13. Ranking de organizaciones según su cumplimiento documental**
```sql
SELECT nombre_organizacion, porcentaje_cumplimiento,
       RANK() OVER (ORDER BY porcentaje_cumplimiento DESC) AS posicion_ranking
FROM vm_tenant_docs_summary;
```

**14. Cumplimiento y diferencia respecto al promedio general**
```sql
SELECT nombre_organizacion, porcentaje_cumplimiento,
       ROUND(porcentaje_cumplimiento - AVG(porcentaje_cumplimiento) OVER (), 2) AS diferencia_respecto_promedio
FROM vm_tenant_docs_summary;
```

**15. Cantidad acumulada de documentos finalizados por organización**
```sql
SELECT organizacion.name AS nombre_organizacion, plantilla_organizacion.assigned_at AS fecha_asignacion,
       SUM(CASE WHEN plantilla_organizacion.status = 'finalizado' THEN 1 ELSE 0 END)
           OVER (PARTITION BY organizacion.id ORDER BY plantilla_organizacion.assigned_at) AS finalizados_acumulados
FROM tenanttemplates AS plantilla_organizacion
JOIN tenants AS organizacion ON organizacion.id = plantilla_organizacion.tenant_id
ORDER BY organizacion.name, plantilla_organizacion.assigned_at;
```

**16. Organizaciones que comparten municipio con distinto tamaño empresarial**
```sql
SELECT organizacion_uno.name AS organizacion_uno, organizacion_dos.name AS organizacion_dos, ciudad.name AS municipio
FROM tenants AS organizacion_uno
JOIN tenants AS organizacion_dos ON organizacion_dos.city_id = organizacion_uno.city_id AND organizacion_dos.id > organizacion_uno.id
JOIN cities AS ciudad ON ciudad.id = organizacion_uno.city_id
WHERE organizacion_uno.tenant_size_id <> organizacion_dos.tenant_size_id;
```

**17. Personas cuyo cargo es ocupado por más personas que el promedio de ocupación en su organización**
```sql
SELECT persona.first_name AS nombre, persona.last_name AS apellido, cargo.description AS cargo
FROM persons AS persona
JOIN positions AS cargo ON cargo.id = persona.position_id
JOIN (
    SELECT position_id, COUNT(*) AS personas_en_cargo
    FROM persons
    GROUP BY position_id
) AS ocupacion_cargo ON ocupacion_cargo.position_id = persona.position_id
JOIN (
    SELECT tenant_id, COUNT(*)::numeric / COUNT(DISTINCT position_id) AS promedio_ocupacion
    FROM persons
    GROUP BY tenant_id
) AS promedio_organizacion ON promedio_organizacion.tenant_id = persona.tenant_id
WHERE ocupacion_cargo.personas_en_cargo > promedio_organizacion.promedio_ocupacion;
```

**18. CTE: personas por organización, filtrando las que superan el promedio**
```sql
WITH personas_por_organizacion AS (
    SELECT organizacion.id AS tenant_id, organizacion.name AS nombre_organizacion, COUNT(persona.id) AS total_personas
    FROM tenants AS organizacion
    LEFT JOIN persons AS persona ON persona.tenant_id = organizacion.id
    GROUP BY organizacion.id, organizacion.name
)
SELECT nombre_organizacion, total_personas
FROM personas_por_organizacion
WHERE total_personas > (SELECT AVG(total_personas) FROM personas_por_organizacion);
```

**19. CTE: consolidar módulos, plantillas y personas por organización**
```sql
WITH resumen_organizacion AS (
    SELECT organizacion.id AS tenant_id, organizacion.name AS nombre_organizacion,
           COUNT(DISTINCT modulo_organizacion.module_id) AS total_modulos,
           COUNT(DISTINCT plantilla_organizacion.id) AS total_plantillas,
           COUNT(DISTINCT persona.id) AS total_personas
    FROM tenants AS organizacion
    LEFT JOIN tenant_modules AS modulo_organizacion ON modulo_organizacion.tenant_id = organizacion.id
    LEFT JOIN tenanttemplates AS plantilla_organizacion ON plantilla_organizacion.tenant_id = organizacion.id
    LEFT JOIN persons AS persona ON persona.tenant_id = organizacion.id
    GROUP BY organizacion.id, organizacion.name
)
SELECT * FROM resumen_organizacion;
```

**20. Organizaciones sin alguna etapa PHVA configurada en sus plantillas**
```sql
SELECT organizacion.name AS nombre_organizacion, etapa_phva.name AS etapa_faltante
FROM tenants AS organizacion
CROSS JOIN phva_stages AS etapa_phva
LEFT JOIN tenanttemplates AS plantilla_organizacion
    ON plantilla_organizacion.tenant_id = organizacion.id
    AND plantilla_organizacion.phva_stage_id = etapa_phva.id
WHERE plantilla_organizacion.id IS NULL
  AND EXISTS (SELECT 1 FROM tenanttemplates WHERE tenant_id = organizacion.id);
```

**21. Última fecha de actualización por organización considerando sus plantillas**
```sql
SELECT organizacion.name AS nombre_organizacion, MAX(plantilla_organizacion.updated_at) AS ultima_actualizacion
FROM tenants AS organizacion
JOIN tenanttemplates AS plantilla_organizacion ON plantilla_organizacion.tenant_id = organizacion.id
GROUP BY organizacion.name;
```

**22. Organizaciones con registros pendientes usando las vistas SST y PESV**
```sql
SELECT COALESCE(resumen_sst.nombre_organizacion, resumen_pesv.nombre_organizacion) AS nombre_organizacion,
       resumen_sst.pendientes AS pendientes_sst,
       resumen_pesv.pendientes AS pendientes_pesv
FROM vm_template_sst_docs_summary AS resumen_sst
FULL JOIN vm_template_pesv_docs_summary AS resumen_pesv ON resumen_pesv.tenant_id = resumen_sst.tenant_id
WHERE COALESCE(resumen_sst.pendientes, 0) > 0 OR COALESCE(resumen_pesv.pendientes, 0) > 0;
```

**23. Informe consolidado por organización**
```sql
SELECT nombre_organizacion, total_documentos, finalizados, borrador, no_iniciados, pendientes, porcentaje_cumplimiento
FROM vm_tenant_docs_summary;
```

**24. Comparar cumplimiento SST vs PESV con diferencia superior a un valor**
```sql
SELECT COALESCE(resumen_sst.nombre_organizacion, resumen_pesv.nombre_organizacion) AS nombre_organizacion,
       resumen_sst.porcentaje_cumplimiento AS cumplimiento_sst,
       resumen_pesv.porcentaje_cumplimiento AS cumplimiento_pesv,
       ABS(COALESCE(resumen_sst.porcentaje_cumplimiento, 0) - COALESCE(resumen_pesv.porcentaje_cumplimiento, 0)) AS diferencia
FROM vm_template_sst_docs_summary AS resumen_sst
FULL JOIN vm_template_pesv_docs_summary AS resumen_pesv ON resumen_pesv.tenant_id = resumen_sst.tenant_id
WHERE ABS(COALESCE(resumen_sst.porcentaje_cumplimiento, 0) - COALESCE(resumen_pesv.porcentaje_cumplimiento, 0)) > 20;
```

**25. Vista que consolide personas, módulos, plantillas y sistemas por organización**
```sql
CREATE VIEW vw_tenant_resumen_general AS
SELECT organizacion.id AS tenant_id, organizacion.name AS nombre_organizacion,
       COUNT(DISTINCT persona.id) AS total_personas,
       COUNT(DISTINCT modulo_organizacion.module_id) AS total_modulos,
       COUNT(DISTINCT plantilla_organizacion.id) AS total_plantillas,
       COUNT(DISTINCT sistema_organizacion.system_id) AS total_sistemas
FROM tenants AS organizacion
LEFT JOIN persons AS persona ON persona.tenant_id = organizacion.id
LEFT JOIN tenant_modules AS modulo_organizacion ON modulo_organizacion.tenant_id = organizacion.id
LEFT JOIN tenanttemplates AS plantilla_organizacion ON plantilla_organizacion.tenant_id = organizacion.id
LEFT JOIN tenantsystems AS sistema_organizacion ON sistema_organizacion.tenant_id = organizacion.id
GROUP BY organizacion.id, organizacion.name;
```

---

## 4. Consultas orientadas a vistas y vistas materializadas ✅

**1. Vista `vw_tenant_persons`**

**Qué hace:** por cada persona registrada, muestra el nombre de su organización, su nombre completo y su cargo, en una sola fila ya lista para reportes.
```sql
CREATE VIEW vw_tenant_persons AS
SELECT organizacion.name AS nombre_organizacion,
       persona.first_name || ' ' || persona.last_name AS nombre_completo_persona,
       cargo.description AS cargo
FROM tenants AS organizacion
JOIN persons AS persona ON persona.tenant_id = organizacion.id
LEFT JOIN positions AS cargo ON cargo.id = persona.position_id;
```
**Cómo comprobarla:** una vez creada, se consulta igual que una tabla:
```sql
SELECT * FROM vw_tenant_persons;
```
Debe devolver una fila por persona (13 en el dataset), con su organización y cargo ya resueltos, sin que tengas que escribir el `JOIN` cada vez.

**2. Vista de información geográfica**

**Qué hace:** resuelve la cadena completa ciudad → departamento → país para cada organización, sin tener que repetir esos 3 `JOIN` cada vez que se necesite la ubicación.
```sql
CREATE VIEW vw_tenant_ubicacion_geografica AS
SELECT organizacion.name AS nombre_organizacion,
       ciudad.name AS municipio,
       departamento.name AS departamento,
       pais.name AS pais
FROM tenants AS organizacion
LEFT JOIN cities AS ciudad ON ciudad.id = organizacion.city_id
LEFT JOIN departments AS departamento ON departamento.id = ciudad.department_id
LEFT JOIN countries AS pais ON pais.id = departamento.country_id;
```
**Cómo comprobarla:**
```sql
SELECT * FROM vw_tenant_ubicacion_geografica;
```
Debe salir una fila por organización (6 en el dataset) mostrando su municipio, departamento y país, por ejemplo "Constructora Andes S.A.S | Medellín | Antioquia | Colombia".

**3. Vista de módulos habilitados por organización y su sistema SST**

**Qué hace:** lista, para cada organización, qué módulos tiene activados y a qué sistema (SG-SST o PESV) pertenece cada uno.
```sql
CREATE VIEW vw_tenant_modulos_sistema AS
SELECT organizacion.name AS nombre_organizacion,
       modulo.title AS nombre_modulo,
       sistema_sst.name AS sistema_sst
FROM tenant_modules AS modulo_organizacion
JOIN tenants AS organizacion ON organizacion.id = modulo_organizacion.tenant_id
JOIN modules AS modulo ON modulo.id = modulo_organizacion.module_id
JOIN type_system_sst AS sistema_sst ON sistema_sst.id = modulo.system_id;
```
**Cómo comprobarla:**
```sql
SELECT * FROM vw_tenant_modulos_sistema ORDER BY nombre_organizacion;
```
"Transportes del Valle Ltda" debe aparecer con "Plan de acción vial" y "Diagnóstico vial" (sistema PESV) además de "Política SST" (sistema SG-SST).

**4. Vista de cantidad de plantillas por organización y etapa PHVA**

**Qué hace:** cuenta cuántas plantillas tiene asignadas cada organización, agrupadas por etapa PHVA (Planear, Hacer, Verificar, Actuar).
```sql
CREATE VIEW vw_tenant_plantillas_por_etapa AS
SELECT organizacion.name AS nombre_organizacion,
       etapa_phva.name AS etapa_phva,
       COUNT(plantilla_organizacion.id) AS total_plantillas
FROM tenanttemplates AS plantilla_organizacion
JOIN tenants AS organizacion ON organizacion.id = plantilla_organizacion.tenant_id
JOIN phva_stages AS etapa_phva ON etapa_phva.id = plantilla_organizacion.phva_stage_id
GROUP BY organizacion.name, etapa_phva.name;
```
**Cómo comprobarla:**
```sql
SELECT * FROM vw_tenant_plantillas_por_etapa ORDER BY nombre_organizacion, etapa_phva;
```
"Constructora Andes S.A.S" debe mostrar las 4 etapas (Planear, Hacer, Verificar, Actuar), porque es la organización que armamos con las 4 etapas completas.

**5. Vista de total de personas por organización y cargo**

**Qué hace:** cuenta cuántas personas ocupan cada cargo, dentro de cada organización.
```sql
CREATE VIEW vw_tenant_personas_por_cargo AS
SELECT organizacion.name AS nombre_organizacion,
       cargo.description AS cargo,
       COUNT(persona.id) AS total_personas
FROM persons AS persona
JOIN tenants AS organizacion ON organizacion.id = persona.tenant_id
JOIN positions AS cargo ON cargo.id = persona.position_id
GROUP BY organizacion.name, cargo.description;
```
**Cómo comprobarla:**
```sql
SELECT * FROM vw_tenant_personas_por_cargo WHERE nombre_organizacion = 'Constructora Andes S.A.S';
```
Debe mostrar "Operario" con 3 personas (Luis, Sofía y Andrés), y "Gerente General" y "Coordinador SST" con 1 cada uno.

**6. Vista materializada de resumen documental por organización**

**Qué hace:** precalcula, por organización, el total de documentos y cuántos están en cada estado (finalizado, borrador, no iniciado, pendiente), junto con el porcentaje de cumplimiento. A diferencia de una vista normal, guarda los datos físicamente y no se recalcula sola (ver punto 7).
```sql
CREATE MATERIALIZED VIEW vm_tenant_docs_summary AS
SELECT organizacion.id AS tenant_id,
       organizacion.name AS nombre_organizacion,
       COUNT(plantilla_organizacion.id) AS total_documentos,
       COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'finalizado') AS finalizados,
       COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'borrador') AS borrador,
       COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'no_iniciado') AS no_iniciados,
       COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'pendiente') AS pendientes,
       ROUND(100.0 * COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'finalizado') / NULLIF(COUNT(plantilla_organizacion.id), 0), 2) AS porcentaje_cumplimiento
FROM tenants AS organizacion
LEFT JOIN tenanttemplates AS plantilla_organizacion ON plantilla_organizacion.tenant_id = organizacion.id
GROUP BY organizacion.id, organizacion.name;
```

Adicionalmente, para las consultas 22 y 24 de la sección 3 se necesitan estas dos vistas materializadas por sistema:

```sql
CREATE MATERIALIZED VIEW vm_template_sst_docs_summary AS
SELECT organizacion.id AS tenant_id, organizacion.name AS nombre_organizacion,
       COUNT(plantilla_organizacion.id) AS total_documentos,
       COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'finalizado') AS finalizados,
       COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'pendiente') AS pendientes,
       ROUND(100.0 * COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'finalizado') / NULLIF(COUNT(plantilla_organizacion.id), 0), 2) AS porcentaje_cumplimiento
FROM tenants AS organizacion
JOIN tenanttemplates AS plantilla_organizacion ON plantilla_organizacion.tenant_id = organizacion.id
JOIN type_system_sst AS sistema_sst ON sistema_sst.id = plantilla_organizacion.system_id AND sistema_sst.name = 'SG-SST'
GROUP BY organizacion.id, organizacion.name;

CREATE MATERIALIZED VIEW vm_template_pesv_docs_summary AS
SELECT organizacion.id AS tenant_id, organizacion.name AS nombre_organizacion,
       COUNT(plantilla_organizacion.id) AS total_documentos,
       COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'finalizado') AS finalizados,
       COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'pendiente') AS pendientes,
       ROUND(100.0 * COUNT(*) FILTER (WHERE plantilla_organizacion.status = 'finalizado') / NULLIF(COUNT(plantilla_organizacion.id), 0), 2) AS porcentaje_cumplimiento
FROM tenants AS organizacion
JOIN tenanttemplates AS plantilla_organizacion ON plantilla_organizacion.tenant_id = organizacion.id
JOIN type_system_sst AS sistema_sst ON sistema_sst.id = plantilla_organizacion.system_id AND sistema_sst.name = 'PESV'
GROUP BY organizacion.id, organizacion.name;
```
**Cómo comprobarlas:**
```sql
SELECT * FROM vm_tenant_docs_summary;
SELECT * FROM vm_template_sst_docs_summary;
SELECT * FROM vm_template_pesv_docs_summary;
```
Cada una debe traer una fila por organización (o solo las que tienen ese sistema, en el caso de las dos últimas) con sus conteos y porcentaje ya calculados. Probado: "Constructora Andes S.A.S" salió con 5 documentos, 3 finalizados, 60% de cumplimiento en `vm_tenant_docs_summary`.

**7. Refrescar la vista materializada y verificar el cambio**

**Qué hace:** demuestra la diferencia clave entre una vista normal y una materializada: la materializada no se actualiza sola cuando cambian los datos base, hay que decírselo explícitamente con `REFRESH MATERIALIZED VIEW`.
```sql
REFRESH MATERIALIZED VIEW vm_tenant_docs_summary;

SELECT * FROM vm_tenant_docs_summary WHERE tenant_id = 1;
```
**Cómo comprobarlo:** se hace en 4 pasos, en este orden:
```sql
-- 1. Consultar el valor actual
SELECT * FROM vm_tenant_docs_summary WHERE tenant_id = 1;

-- 2. Insertar un documento nuevo
INSERT INTO tenanttemplates (tenant_id, template_id, system_id, phva_stage_id, format_id, status)
VALUES (1, 6, 2, 1, 4, 'finalizado');

-- 3. Consultar SIN refrescar (todavía debe mostrar el valor viejo)
SELECT * FROM vm_tenant_docs_summary WHERE tenant_id = 1;

-- 4. Refrescar y consultar de nuevo (ya debe reflejar el cambio)
REFRESH MATERIALIZED VIEW vm_tenant_docs_summary;
SELECT * FROM vm_tenant_docs_summary WHERE tenant_id = 1;
```
Lo probé así: antes del `INSERT`, Constructora Andes mostraba 5 documentos / 60%. Después del `INSERT` pero antes del `REFRESH`, seguía en 5 / 60% (dato desactualizado). Después del `REFRESH`, pasó a 6 documentos / 66.67% (ya actualizado).

**8. Índices recomendados para la vista materializada**

**Qué hace:** acelera las consultas que filtran por organización o que ordenan/comparan por porcentaje de cumplimiento, en lugar de recorrer toda la vista fila por fila.
```sql
CREATE UNIQUE INDEX idx_vm_tenant_docs_summary_tenant_id ON vm_tenant_docs_summary(tenant_id);
CREATE INDEX idx_vm_tenant_docs_summary_cumplimiento ON vm_tenant_docs_summary(porcentaje_cumplimiento);
```
- `tenant_id` como índice único: cada organización aparece una sola vez y casi toda consulta de seguimiento filtra por una organización puntual. Además habilita `REFRESH MATERIALIZED VIEW CONCURRENTLY`.
- `porcentaje_cumplimiento`: se usa constantemente para ordenar (ranking) o comparar contra el promedio.

**Cómo comprobar que el índice existe y se usa:**
```sql
\d vm_tenant_docs_summary

EXPLAIN SELECT * FROM vm_tenant_docs_summary WHERE tenant_id = 1;
```
El `\d` debe listar los dos índices creados, y el `EXPLAIN` debe mostrar `Index Scan using idx_vm_tenant_docs_summary_tenant_id` en vez de un recorrido secuencial (`Seq Scan`) cuando la tabla crezca lo suficiente.

---

## 5. Procedimientos almacenados ✅

**Orden de creación:** las funciones de la sección 6 deben crearse primero, porque el procedimiento 12 y el trigger 11 dependen de `fn_porcentaje_cumplimiento`.

**1. Registrar organización (valida NIT duplicado)**
```sql
CREATE OR REPLACE PROCEDURE sp_registrar_organizacion(
    p_name VARCHAR,
    p_nit VARCHAR,
    p_contact_email VARCHAR,
    p_contact_phone VARCHAR,
    p_tenant_size_id INTEGER,
    p_city_id INTEGER
)
LANGUAGE plpgsql
AS $$
BEGIN
    IF EXISTS (SELECT 1 FROM tenants WHERE nit = p_nit) THEN
        RAISE EXCEPTION 'Ya existe una organización registrada con el NIT %', p_nit;
    END IF;

    INSERT INTO tenants (name, nit, contact_email, contact_phone, tenant_size_id, city_id)
    VALUES (p_name, p_nit, p_contact_email, p_contact_phone, p_tenant_size_id, p_city_id);
END;
$$;
```
Prueba: `CALL sp_registrar_organizacion('Prueba Uno S.A.S', '900000001-1', 'prueba@uno.com', '3000000001', 1, 1);` y luego repetirla con el mismo NIT lanza `ERROR: Ya existe una organización registrada con el NIT 900000001-1`.

**2. Registrar persona y asociarla a organización y cargo**
```sql
CREATE OR REPLACE PROCEDURE sp_registrar_persona(
    p_tenant_id INTEGER,
    p_position_id INTEGER,
    p_first_name VARCHAR,
    p_last_name VARCHAR,
    p_email VARCHAR
)
LANGUAGE plpgsql
AS $$
BEGIN
    IF NOT EXISTS (SELECT 1 FROM tenants WHERE id = p_tenant_id) THEN
        RAISE EXCEPTION 'La organización % no existe', p_tenant_id;
    END IF;

    IF p_position_id IS NOT NULL AND NOT EXISTS (
        SELECT 1 FROM positions WHERE id = p_position_id AND tenant_id = p_tenant_id
    ) THEN
        RAISE EXCEPTION 'El cargo % no pertenece a la organización %', p_position_id, p_tenant_id;
    END IF;

    INSERT INTO persons (tenant_id, position_id, first_name, last_name, email)
    VALUES (p_tenant_id, p_position_id, p_first_name, p_last_name, p_email);
END;
$$;
```

**3. Cambiar estado activa/inactiva**
```sql
CREATE OR REPLACE PROCEDURE sp_cambiar_estado_organizacion(p_tenant_id INTEGER, p_nuevo_estado BOOLEAN)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE tenants SET status = p_nuevo_estado WHERE id = p_tenant_id;

    IF NOT FOUND THEN
        RAISE EXCEPTION 'No existe la organización %', p_tenant_id;
    END IF;
END;
$$;
```

**4. Asignar módulo evitando duplicados**
```sql
CREATE OR REPLACE PROCEDURE sp_asignar_modulo(p_tenant_id INTEGER, p_module_id INTEGER)
LANGUAGE plpgsql
AS $$
BEGIN
    IF EXISTS (SELECT 1 FROM tenant_modules WHERE tenant_id = p_tenant_id AND module_id = p_module_id) THEN
        RAISE EXCEPTION 'El módulo % ya está asignado a la organización %', p_module_id, p_tenant_id;
    END IF;

    INSERT INTO tenant_modules (tenant_id, module_id) VALUES (p_tenant_id, p_module_id);
END;
$$;
```

**5. Habilitar sistema SST**
```sql
CREATE OR REPLACE PROCEDURE sp_habilitar_sistema(p_tenant_id INTEGER, p_system_id INTEGER)
LANGUAGE plpgsql
AS $$
BEGIN
    IF EXISTS (SELECT 1 FROM tenantsystems WHERE tenant_id = p_tenant_id AND system_id = p_system_id) THEN
        RAISE EXCEPTION 'El sistema % ya está habilitado para la organización %', p_system_id, p_tenant_id;
    END IF;

    INSERT INTO tenantsystems (tenant_id, system_id) VALUES (p_tenant_id, p_system_id);
END;
$$;
```

**6. Asignar plantilla a organización (sistema, etapa PHVA y formato)**
```sql
CREATE OR REPLACE PROCEDURE sp_asignar_plantilla(
    p_tenant_id INTEGER,
    p_template_id INTEGER,
    p_system_id INTEGER,
    p_phva_stage_id INTEGER,
    p_format_id INTEGER
)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO tenanttemplates (tenant_id, template_id, system_id, phva_stage_id, format_id)
    VALUES (p_tenant_id, p_template_id, p_system_id, p_phva_stage_id, p_format_id);
END;
$$;
```

**7. Cambiar el cargo de una persona (validando que el cargo sea de su misma organización)**
```sql
CREATE OR REPLACE PROCEDURE sp_cambiar_cargo_persona(p_person_id INTEGER, p_new_position_id INTEGER)
LANGUAGE plpgsql
AS $$
DECLARE
    v_tenant_id INTEGER;
BEGIN
    SELECT tenant_id INTO v_tenant_id FROM persons WHERE id = p_person_id;

    IF v_tenant_id IS NULL THEN
        RAISE EXCEPTION 'No existe la persona %', p_person_id;
    END IF;

    IF NOT EXISTS (SELECT 1 FROM positions WHERE id = p_new_position_id AND tenant_id = v_tenant_id) THEN
        RAISE EXCEPTION 'El cargo % no pertenece a la organización de la persona', p_new_position_id;
    END IF;

    UPDATE persons SET position_id = p_new_position_id WHERE id = p_person_id;
END;
$$;
```

**8. Trasladar persona de una organización a otra**
```sql
CREATE OR REPLACE PROCEDURE sp_trasladar_persona(p_person_id INTEGER, p_new_tenant_id INTEGER, p_new_position_id INTEGER)
LANGUAGE plpgsql
AS $$
BEGIN
    IF NOT EXISTS (SELECT 1 FROM tenants WHERE id = p_new_tenant_id) THEN
        RAISE EXCEPTION 'La organización destino % no existe', p_new_tenant_id;
    END IF;

    IF NOT EXISTS (SELECT 1 FROM positions WHERE id = p_new_position_id AND tenant_id = p_new_tenant_id) THEN
        RAISE EXCEPTION 'El cargo % no pertenece a la organización destino %', p_new_position_id, p_new_tenant_id;
    END IF;

    UPDATE persons
    SET tenant_id = p_new_tenant_id,
        position_id = p_new_position_id
    WHERE id = p_person_id;

    IF NOT FOUND THEN
        RAISE EXCEPTION 'No existe la persona %', p_person_id;
    END IF;
END;
$$;
```

**9. Deshabilitar módulos de una organización inactiva**
```sql
CREATE OR REPLACE PROCEDURE sp_deshabilitar_modulos_organizacion_inactiva(p_tenant_id INTEGER)
LANGUAGE plpgsql
AS $$
DECLARE
    v_status BOOLEAN;
BEGIN
    SELECT status INTO v_status FROM tenants WHERE id = p_tenant_id;

    IF v_status IS NULL THEN
        RAISE EXCEPTION 'No existe la organización %', p_tenant_id;
    END IF;

    IF v_status = TRUE THEN
        RAISE EXCEPTION 'La organización % está activa, no se pueden deshabilitar sus módulos', p_tenant_id;
    END IF;

    DELETE FROM tenant_modules WHERE tenant_id = p_tenant_id;
END;
$$;
```

**10. Eliminar asignación de módulo validando dependientes**

Se valida contra `evaluations`, ya que esa tabla depende directamente de la combinación organización + módulo.
```sql
CREATE OR REPLACE PROCEDURE sp_eliminar_asignacion_modulo(p_tenant_id INTEGER, p_module_id INTEGER)
LANGUAGE plpgsql
AS $$
BEGIN
    IF EXISTS (
        SELECT 1 FROM evaluations WHERE tenant_id = p_tenant_id AND module_id = p_module_id
    ) THEN
        RAISE EXCEPTION 'No se puede eliminar: la organización % tiene evaluaciones registradas para el módulo %', p_tenant_id, p_module_id;
    END IF;

    DELETE FROM tenant_modules WHERE tenant_id = p_tenant_id AND module_id = p_module_id;

    IF NOT FOUND THEN
        RAISE EXCEPTION 'La organización % no tenía asignado el módulo %', p_tenant_id, p_module_id;
    END IF;
END;
$$;
```

**11. Total de plantillas de una organización con `RAISE NOTICE`**
```sql
CREATE OR REPLACE PROCEDURE sp_total_plantillas_organizacion(p_tenant_id INTEGER)
LANGUAGE plpgsql
AS $$
DECLARE
    v_total INTEGER;
BEGIN
    SELECT COUNT(*) INTO v_total FROM tenanttemplates WHERE tenant_id = p_tenant_id;
    RAISE NOTICE 'La organización % tiene % plantillas asignadas', p_tenant_id, v_total;
END;
$$;
```
Prueba: `CALL sp_total_plantillas_organizacion(1);` → `NOTICE: La organización 1 tiene 6 plantillas asignadas`.

**12. Porcentaje de cumplimiento a partir de finalizados y pendientes (parámetro de salida)**
```sql
CREATE OR REPLACE PROCEDURE sp_porcentaje_cumplimiento_organizacion(
    p_tenant_id INTEGER,
    OUT p_porcentaje NUMERIC
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_finalizados INTEGER;
    v_pendientes INTEGER;
BEGIN
    SELECT COUNT(*) FILTER (WHERE status = 'finalizado'),
           COUNT(*) FILTER (WHERE status = 'pendiente')
    INTO v_finalizados, v_pendientes
    FROM tenanttemplates
    WHERE tenant_id = p_tenant_id;

    IF (v_finalizados + v_pendientes) = 0 THEN
        p_porcentaje := 0;
    ELSE
        p_porcentaje := ROUND(100.0 * v_finalizados / (v_finalizados + v_pendientes), 2);
    END IF;
END;
$$;
```
Se invoca como `CALL sp_porcentaje_cumplimiento_organizacion(1, NULL);` (el `NULL` es obligatorio como marcador del parámetro `OUT`).

**13. Total de documentos por organización y etapa PHVA (parámetro de salida)**
```sql
CREATE OR REPLACE PROCEDURE sp_total_documentos_por_etapa(
    p_tenant_id INTEGER,
    p_phva_stage_id INTEGER,
    OUT p_total INTEGER
)
LANGUAGE plpgsql
AS $$
BEGIN
    SELECT COUNT(*) INTO p_total
    FROM tenanttemplates
    WHERE tenant_id = p_tenant_id AND phva_stage_id = p_phva_stage_id;
END;
$$;
```

**14. Modificar datos de contacto y registrar fecha de actualización**
```sql
CREATE OR REPLACE PROCEDURE sp_actualizar_contacto_organizacion(
    p_tenant_id INTEGER,
    p_contact_email VARCHAR,
    p_contact_phone VARCHAR
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE tenants
    SET contact_email = p_contact_email,
        contact_phone = p_contact_phone,
        updated_at = NOW()
    WHERE id = p_tenant_id;

    IF NOT FOUND THEN
        RAISE EXCEPTION 'No existe la organización %', p_tenant_id;
    END IF;
END;
$$;
```

**15. Manejo de excepciones al asignar plantillas**
```sql
CREATE OR REPLACE PROCEDURE sp_asignar_plantilla_segura(
    p_tenant_id INTEGER,
    p_template_id INTEGER,
    p_system_id INTEGER,
    p_phva_stage_id INTEGER,
    p_format_id INTEGER
)
LANGUAGE plpgsql
AS $$
BEGIN
    BEGIN
        INSERT INTO tenanttemplates (tenant_id, template_id, system_id, phva_stage_id, format_id)
        VALUES (p_tenant_id, p_template_id, p_system_id, p_phva_stage_id, p_format_id);

        RAISE NOTICE 'Plantilla % asignada correctamente a la organización %', p_template_id, p_tenant_id;
    EXCEPTION
        WHEN foreign_key_violation THEN
            RAISE NOTICE 'Error: alguno de los identificadores enviados no existe (organización, plantilla, sistema, etapa o formato)';
        WHEN check_violation THEN
            RAISE NOTICE 'Error: el estado del documento no cumple con los valores permitidos';
        WHEN OTHERS THEN
            RAISE NOTICE 'Error inesperado al asignar la plantilla: %', SQLERRM;
    END;
END;
$$;
```
Prueba con organización inexistente (999): en vez de abortar la sesión con un error crudo, imprime `NOTICE: Error: alguno de los identificadores enviados no existe...`.

---

## 6. Funciones almacenadas ✅

**1. Total de personas por organización**
```sql
CREATE OR REPLACE FUNCTION fn_total_personas_organizacion(p_tenant_id INTEGER)
RETURNS INTEGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_total INTEGER;
BEGIN
    SELECT COUNT(*) INTO v_total FROM persons WHERE tenant_id = p_tenant_id;
    RETURN v_total;
END;
$$;
```
Uso: `SELECT fn_total_personas_organizacion(1);`

**2. Porcentaje de cumplimiento documental**
```sql
CREATE OR REPLACE FUNCTION fn_porcentaje_cumplimiento(p_tenant_id INTEGER)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
DECLARE
    v_total INTEGER;
    v_finalizados INTEGER;
BEGIN
    SELECT COUNT(*), COUNT(*) FILTER (WHERE status = 'finalizado')
    INTO v_total, v_finalizados
    FROM tenanttemplates WHERE tenant_id = p_tenant_id;

    IF v_total = 0 THEN
        RETURN 0;
    END IF;

    RETURN ROUND(100.0 * v_finalizados / v_total, 2);
END;
$$;
```

**3. ¿Organización tiene un módulo habilitado? (booleano)**
```sql
CREATE OR REPLACE FUNCTION fn_tiene_modulo_habilitado(p_tenant_id INTEGER, p_module_id INTEGER)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN EXISTS (
        SELECT 1 FROM tenant_modules WHERE tenant_id = p_tenant_id AND module_id = p_module_id
    );
END;
$$;
```

**4. Nombre completo de una persona**
```sql
CREATE OR REPLACE FUNCTION fn_nombre_completo_persona(p_person_id INTEGER)
RETURNS VARCHAR
LANGUAGE plpgsql
AS $$
DECLARE
    v_nombre_completo VARCHAR;
BEGIN
    SELECT first_name || ' ' || last_name INTO v_nombre_completo
    FROM persons WHERE id = p_person_id;

    RETURN v_nombre_completo;
END;
$$;
```

**5. Total de plantillas por organización y etapa PHVA**
```sql
CREATE OR REPLACE FUNCTION fn_total_plantillas_por_etapa(p_tenant_id INTEGER, p_phva_stage_id INTEGER)
RETURNS INTEGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_total INTEGER;
BEGIN
    SELECT COUNT(*) INTO v_total
    FROM tenanttemplates
    WHERE tenant_id = p_tenant_id AND phva_stage_id = p_phva_stage_id;

    RETURN v_total;
END;
$$;
```

**6. Función tabular: módulos habilitados para una organización**
```sql
CREATE OR REPLACE FUNCTION fn_modulos_organizacion(p_tenant_id INTEGER)
RETURNS TABLE(modulo_id INTEGER, nombre_modulo VARCHAR, sistema_sst VARCHAR)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT modulo.id, modulo.title, sistema_sst.name
    FROM tenant_modules AS modulo_organizacion
    JOIN modules AS modulo ON modulo.id = modulo_organizacion.module_id
    JOIN type_system_sst AS sistema_sst ON sistema_sst.id = modulo.system_id
    WHERE modulo_organizacion.tenant_id = p_tenant_id;
END;
$$;
```
Uso: `SELECT * FROM fn_modulos_organizacion(1);`

**7. Función tabular: personas de una organización con sus cargos**

Nota: `first_name || ' ' || last_name` produce el tipo `text`, por eso la columna de salida se declara como `TEXT` (no `VARCHAR`) para que coincida exactamente.
```sql
CREATE OR REPLACE FUNCTION fn_personas_organizacion(p_tenant_id INTEGER)
RETURNS TABLE(persona_id INTEGER, nombre_completo TEXT, cargo VARCHAR)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT persona.id, persona.first_name || ' ' || persona.last_name, cargo.description
    FROM persons AS persona
    LEFT JOIN positions AS cargo ON cargo.id = persona.position_id
    WHERE persona.tenant_id = p_tenant_id;
END;
$$;
```
Uso: `SELECT * FROM fn_personas_organizacion(1);`

**8. Clasificar nivel de cumplimiento (bajo, medio, alto)**
```sql
CREATE OR REPLACE FUNCTION fn_clasificar_cumplimiento(p_tenant_id INTEGER)
RETURNS VARCHAR
LANGUAGE plpgsql
AS $$
DECLARE
    v_porcentaje NUMERIC;
BEGIN
    v_porcentaje := fn_porcentaje_cumplimiento(p_tenant_id);

    IF v_porcentaje < 40 THEN
        RETURN 'Bajo';
    ELSIF v_porcentaje < 70 THEN
        RETURN 'Medio';
    ELSE
        RETURN 'Alto';
    END IF;
END;
$$;
```

---

## 7. Triggers ✅

**Nota importante:** los triggers 1 y 2 (actualizar `updated_at` en `tenants` y `persons`) ya se habían creado en la sección de tablas, reutilizando la función genérica `set_updated_at()`. El trigger 14 (fecha + usuario responsable en `tenanttemplates`) reemplaza al trigger genérico de `updated_at` que existía en esa tabla, porque hace ambas tareas a la vez (ver detalle más abajo). Por eso el trigger 7 no se implementa por separado: queda cubierto por el 14.

**1 y 2. `updated_at` automático (ya creados)**

**Qué hacen:** cada vez que se modifica una fila de `tenants` o de `persons`, ponen automáticamente `updated_at = NOW()`, sin que el desarrollador tenga que acordarse de hacerlo en cada `UPDATE`.
```sql
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_tenants_updated_at
BEFORE UPDATE ON tenants
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

CREATE TRIGGER trg_persons_updated_at
BEFORE UPDATE ON persons
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```
**Cómo comprobarlos:**
```sql
UPDATE tenants SET contact_phone = '3000000000' WHERE id = 2;
SELECT id, contact_phone, updated_at FROM tenants WHERE id = 2;

UPDATE persons SET last_name = 'Ramírez Actualizado' WHERE id = 1;
SELECT id, last_name, updated_at FROM persons WHERE id = 1;
```
En ambos casos `updated_at` debe cambiar al momento exacto del `UPDATE`, aunque la instrucción no haya tocado esa columna explícitamente.

**3. Impedir registrar persona en organización inactiva**

**Qué hace:** antes de insertar una persona, revisa si la organización a la que se quiere asociar está inactiva; si lo está, cancela la operación.
```sql
CREATE OR REPLACE FUNCTION fn_validar_organizacion_activa_persona()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_status BOOLEAN;
BEGIN
    SELECT status INTO v_status FROM tenants WHERE id = NEW.tenant_id;

    IF v_status = FALSE THEN
        RAISE EXCEPTION 'No se puede registrar una persona en una organización inactiva (tenant_id = %)', NEW.tenant_id;
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_persons_organizacion_activa
BEFORE INSERT ON persons
FOR EACH ROW EXECUTE FUNCTION fn_validar_organizacion_activa_persona();
```
**Cómo comprobarlo:**
```sql
-- Textiles Bogotá (tenant_id = 4) está inactiva → debe fallar
INSERT INTO persons (tenant_id, first_name, last_name, email) VALUES (4, 'X', 'Y', 'x.y@test.com');
```
Debe lanzar: `ERROR: No se puede registrar una persona en una organización inactiva (tenant_id = 4)`.

**4. Impedir asignar un módulo ya asignado**

**Qué hace:** antes de insertar una fila en `tenant_modules`, revisa si esa combinación organización + módulo ya existe, y si es así, cancela la operación con un mensaje claro (en vez de dejar que falle con el error genérico de la restricción `UNIQUE`).
```sql
CREATE OR REPLACE FUNCTION fn_validar_modulo_no_duplicado()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF EXISTS (
        SELECT 1 FROM tenant_modules WHERE tenant_id = NEW.tenant_id AND module_id = NEW.module_id
    ) THEN
        RAISE EXCEPTION 'El módulo % ya se encuentra asignado a la organización %', NEW.module_id, NEW.tenant_id;
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_tenant_modules_no_duplicados
BEFORE INSERT ON tenant_modules
FOR EACH ROW EXECUTE FUNCTION fn_validar_modulo_no_duplicado();
```
**Cómo comprobarlo:**
```sql
-- Constructora Andes (tenant_id = 1) ya tiene el módulo 1 asignado → debe fallar
INSERT INTO tenant_modules (tenant_id, module_id) VALUES (1, 1);
```
Debe lanzar: `ERROR: El módulo 1 ya se encuentra asignado a la organización 1`.

**5. Impedir asignar plantillas a organizaciones inactivas**

**Qué hace:** igual que el trigger 3, pero para `tenanttemplates`: si la organización está inactiva, no deja crear el documento.
```sql
CREATE OR REPLACE FUNCTION fn_validar_organizacion_activa_plantilla()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_status BOOLEAN;
BEGIN
    SELECT status INTO v_status FROM tenants WHERE id = NEW.tenant_id;

    IF v_status = FALSE THEN
        RAISE EXCEPTION 'No se pueden asignar plantillas a una organización inactiva (tenant_id = %)', NEW.tenant_id;
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_tenanttemplates_organizacion_activa
BEFORE INSERT ON tenanttemplates
FOR EACH ROW EXECUTE FUNCTION fn_validar_organizacion_activa_plantilla();
```
**Cómo comprobarlo:**
```sql
-- Textiles Bogotá (tenant_id = 4) está inactiva → debe fallar
INSERT INTO tenanttemplates (tenant_id, template_id, system_id, phva_stage_id, format_id) VALUES (4, 1, 1, 1, 1);
```
Debe lanzar: `ERROR: No se pueden asignar plantillas a una organización inactiva (tenant_id = 4)`.

**6. Validar que el cargo de una persona pertenezca a su misma organización**

**Qué hace:** al insertar o actualizar una persona, revisa que el `position_id` que se le asigna pertenezca a la misma organización de esa persona (no al cargo de otra empresa).
```sql
CREATE OR REPLACE FUNCTION fn_validar_cargo_misma_organizacion()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_tenant_cargo INTEGER;
BEGIN
    IF NEW.position_id IS NOT NULL THEN
        SELECT tenant_id INTO v_tenant_cargo FROM positions WHERE id = NEW.position_id;

        IF v_tenant_cargo IS DISTINCT FROM NEW.tenant_id THEN
            RAISE EXCEPTION 'El cargo % no pertenece a la organización % de la persona', NEW.position_id, NEW.tenant_id;
        END IF;
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_persons_cargo_misma_organizacion
BEFORE INSERT OR UPDATE ON persons
FOR EACH ROW EXECUTE FUNCTION fn_validar_cargo_misma_organizacion();
```
**Cómo comprobarlo:**
```sql
-- Persona 1 (Carlos, de la organización 1) intenta tomar el cargo 4 (que es de la organización 2) → debe fallar
UPDATE persons SET position_id = 4 WHERE id = 1;
```
Debe lanzar: `ERROR: El cargo 4 no pertenece a la organización 1 de la persona`.

**7. Fecha de actualización al modificar una plantilla asignada**

Cubierto por el trigger 14 (más abajo), que registra `updated_at` y `updated_by` en un solo trigger.

**8. Impedir eliminar organización con personas asociadas**

**Qué hace:** antes de borrar una organización, revisa si todavía tiene personas registradas; si las tiene, no deja eliminarla (protege contra pérdida accidental de datos relacionados).
```sql
CREATE OR REPLACE FUNCTION fn_impedir_eliminar_organizacion_con_personas()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF EXISTS (SELECT 1 FROM persons WHERE tenant_id = OLD.id) THEN
        RAISE EXCEPTION 'No se puede eliminar la organización %: todavía tiene personas asociadas', OLD.id;
    END IF;

    RETURN OLD;
END;
$$;

CREATE TRIGGER trg_tenants_no_eliminar_con_personas
BEFORE DELETE ON tenants
FOR EACH ROW EXECUTE FUNCTION fn_impedir_eliminar_organizacion_con_personas();
```
**Cómo comprobarlo:**
```sql
-- Constructora Andes (tenant_id = 1) tiene 5 personas → debe fallar
DELETE FROM tenants WHERE id = 1;
```
Debe lanzar: `ERROR: No se puede eliminar la organización 1: todavía tiene personas asociadas`.

**9. Impedir eliminar sistema SST en uso**

**Qué hace:** antes de borrar un registro de `type_system_sst`, revisa si alguna organización lo tiene habilitado (en `tenantsystems`); si es así, no deja eliminarlo.
```sql
CREATE OR REPLACE FUNCTION fn_impedir_eliminar_sistema_en_uso()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF EXISTS (SELECT 1 FROM tenantsystems WHERE system_id = OLD.id) THEN
        RAISE EXCEPTION 'No se puede eliminar el sistema %: existen organizaciones que lo tienen habilitado', OLD.name;
    END IF;

    RETURN OLD;
END;
$$;

CREATE TRIGGER trg_type_system_sst_no_eliminar_en_uso
BEFORE DELETE ON type_system_sst
FOR EACH ROW EXECUTE FUNCTION fn_impedir_eliminar_sistema_en_uso();
```
**Cómo comprobarlo:**
```sql
-- SG-SST (id = 1) está habilitado para varias organizaciones → debe fallar
DELETE FROM type_system_sst WHERE id = 1;
```
Debe lanzar: `ERROR: No se puede eliminar el sistema SG-SST: existen organizaciones que lo tienen habilitado`.

**10. Impedir eliminar módulo asignado**

**Qué hace:** antes de borrar un módulo, revisa si alguna organización lo tiene asignado (en `tenant_modules`); si es así, no deja eliminarlo.
```sql
CREATE OR REPLACE FUNCTION fn_impedir_eliminar_modulo_asignado()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF EXISTS (SELECT 1 FROM tenant_modules WHERE module_id = OLD.id) THEN
        RAISE EXCEPTION 'No se puede eliminar el módulo %: está asignado a una o más organizaciones', OLD.title;
    END IF;

    RETURN OLD;
END;
$$;

CREATE TRIGGER trg_modules_no_eliminar_asignados
BEFORE DELETE ON modules
FOR EACH ROW EXECUTE FUNCTION fn_impedir_eliminar_modulo_asignado();
```
**Cómo comprobarlo:**
```sql
-- El módulo 1 (Política SST) está asignado a varias organizaciones → debe fallar
DELETE FROM modules WHERE id = 1;
```
Debe lanzar: `ERROR: No se puede eliminar el módulo Política SST: está asignado a una o más organizaciones`.

**11. Validar que el porcentaje de cumplimiento esté entre 0 y 100**

**Qué hace:** es una validación de "sanidad" (defensive programming): cada vez que se inserta o actualiza un documento, recalcula el cumplimiento de esa organización y verifica que el número tenga sentido (entre 0 y 100). En condiciones normales nunca debería dispararse, pero protege contra errores de cálculo si la fórmula cambia en el futuro.

Se ejecuta después de insertar o actualizar un documento, recalcula el cumplimiento de la organización con `fn_porcentaje_cumplimiento` (sección 6) y verifica que el resultado esté en el rango permitido. Requiere que la función de la sección 6 ya exista.
```sql
CREATE OR REPLACE FUNCTION fn_validar_rango_cumplimiento()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_porcentaje NUMERIC;
BEGIN
    v_porcentaje := fn_porcentaje_cumplimiento(NEW.tenant_id);

    IF v_porcentaje < 0 OR v_porcentaje > 100 THEN
        RAISE EXCEPTION 'El porcentaje de cumplimiento calculado (%) está fuera del rango permitido (0-100) para la organización %', v_porcentaje, NEW.tenant_id;
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_tenanttemplates_validar_cumplimiento
AFTER INSERT OR UPDATE ON tenanttemplates
FOR EACH ROW EXECUTE FUNCTION fn_validar_rango_cumplimiento();
```
**Cómo comprobarlo:**
```sql
-- Caso normal: no debe fallar, porque el resultado siempre cae entre 0 y 100
INSERT INTO tenanttemplates (tenant_id, template_id, system_id, phva_stage_id, format_id, status)
VALUES (5, 1, 1, 1, 1, 'finalizado');
```
Como el cálculo interno de `fn_porcentaje_cumplimiento` siempre produce un valor entre 0 y 100, este `INSERT` simplemente funciona sin error — lo que demuestra que la validación pasa correctamente en el caso normal (no hay forma de forzar un valor fuera de rango sin romper la fórmula misma).

**12. Auditar modificaciones a los datos principales de una organización**

**Qué hace:** cada vez que se actualiza una organización, compara campo por campo (`name`, `nit`, `contact_email`, `contact_phone`, `tenant_size_id`, `city_id`) el valor anterior contra el nuevo, y guarda una fila en `tenant_audit` por cada campo que realmente haya cambiado.
```sql
CREATE OR REPLACE FUNCTION fn_auditar_cambios_organizacion()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.name IS DISTINCT FROM OLD.name THEN
        INSERT INTO tenant_audit (tenant_id, field_name, old_value, new_value, operation, changed_by)
        VALUES (OLD.id, 'name', OLD.name, NEW.name, 'UPDATE', CURRENT_USER);
    END IF;

    IF NEW.nit IS DISTINCT FROM OLD.nit THEN
        INSERT INTO tenant_audit (tenant_id, field_name, old_value, new_value, operation, changed_by)
        VALUES (OLD.id, 'nit', OLD.nit, NEW.nit, 'UPDATE', CURRENT_USER);
    END IF;

    IF NEW.contact_email IS DISTINCT FROM OLD.contact_email THEN
        INSERT INTO tenant_audit (tenant_id, field_name, old_value, new_value, operation, changed_by)
        VALUES (OLD.id, 'contact_email', OLD.contact_email, NEW.contact_email, 'UPDATE', CURRENT_USER);
    END IF;

    IF NEW.contact_phone IS DISTINCT FROM OLD.contact_phone THEN
        INSERT INTO tenant_audit (tenant_id, field_name, old_value, new_value, operation, changed_by)
        VALUES (OLD.id, 'contact_phone', OLD.contact_phone, NEW.contact_phone, 'UPDATE', CURRENT_USER);
    END IF;

    IF NEW.tenant_size_id IS DISTINCT FROM OLD.tenant_size_id THEN
        INSERT INTO tenant_audit (tenant_id, field_name, old_value, new_value, operation, changed_by)
        VALUES (OLD.id, 'tenant_size_id', OLD.tenant_size_id::TEXT, NEW.tenant_size_id::TEXT, 'UPDATE', CURRENT_USER);
    END IF;

    IF NEW.city_id IS DISTINCT FROM OLD.city_id THEN
        INSERT INTO tenant_audit (tenant_id, field_name, old_value, new_value, operation, changed_by)
        VALUES (OLD.id, 'city_id', OLD.city_id::TEXT, NEW.city_id::TEXT, 'UPDATE', CURRENT_USER);
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_tenants_auditar_cambios
AFTER UPDATE ON tenants
FOR EACH ROW EXECUTE FUNCTION fn_auditar_cambios_organizacion();
```
**Cómo comprobarlo:**
```sql
UPDATE tenants SET name = 'Transportes del Valle S.A.S' WHERE id = 2;
SELECT * FROM tenant_audit WHERE tenant_id = 2;
```
Debe aparecer una fila con `field_name = 'name'`, `old_value = 'Transportes del Valle Ltda'` y `new_value = 'Transportes del Valle S.A.S'`.

**13. Auditar específicamente el cambio de estado (valor anterior y nuevo)**

**Qué hace:** es una auditoría dedicada solo al campo `status` (activa/inactiva), separada de la auditoría general del trigger 12, con su propia etiqueta de operación (`CAMBIO_ESTADO`) para poder filtrarla fácilmente.
```sql
CREATE OR REPLACE FUNCTION fn_auditar_cambio_estado_organizacion()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.status IS DISTINCT FROM OLD.status THEN
        INSERT INTO tenant_audit (tenant_id, field_name, old_value, new_value, operation, changed_by)
        VALUES (OLD.id, 'status', OLD.status::TEXT, NEW.status::TEXT, 'CAMBIO_ESTADO', CURRENT_USER);
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_tenants_auditar_estado
AFTER UPDATE ON tenants
FOR EACH ROW EXECUTE FUNCTION fn_auditar_cambio_estado_organizacion();
```
**Cómo comprobarlo:**
```sql
UPDATE tenants SET status = FALSE WHERE id = 3;
SELECT * FROM tenant_audit WHERE tenant_id = 3 AND field_name = 'status';
```
Debe aparecer `old_value = 'true'`, `new_value = 'false'`, `operation = 'CAMBIO_ESTADO'`.

**14. Registrar fecha y usuario responsable cuando se modifica una plantilla**

Reemplaza al trigger genérico de `updated_at` en `tenanttemplates` (por eso primero se hace `DROP TRIGGER IF EXISTS`), ya que hace ese trabajo y además registra quién hizo el cambio. El "usuario responsable" se lee de una variable de sesión (`app.current_person_id`) que la aplicación debe establecer antes del `UPDATE`.
```sql
DROP TRIGGER IF EXISTS trg_tenanttemplates_updated_at ON tenanttemplates;

CREATE OR REPLACE FUNCTION fn_registrar_modificacion_plantilla()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.updated_at := NOW();

    IF current_setting('app.current_person_id', true) IS NOT NULL
       AND current_setting('app.current_person_id', true) <> '' THEN
        NEW.updated_by := current_setting('app.current_person_id', true)::INTEGER;
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_tenanttemplates_registrar_modificacion
BEFORE UPDATE ON tenanttemplates
FOR EACH ROW EXECUTE FUNCTION fn_registrar_modificacion_plantilla();
```
**Cómo comprobarlo:**
```sql
SET app.current_person_id = '2';
UPDATE tenanttemplates SET status = 'finalizado' WHERE id = 3;
SELECT id, status, updated_at, updated_by FROM tenanttemplates WHERE id = 3;
```
Debe salir `updated_by = 2` y `updated_at` con la hora exacta del `UPDATE`, sin que la instrucción haya mencionado esas dos columnas.

**15. Limpiar bloqueos de edición vencidos**

**Qué hace:** cada vez que alguien va a crear un nuevo bloqueo de edición, aprovecha ese momento para marcar como inactivos (`active = FALSE`) todos los bloqueos que ya vencieron (limpieza "oportunista": no corre solo, se dispara con la siguiente inserción).
```sql
CREATE OR REPLACE FUNCTION fn_limpiar_bloqueos_vencidos()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE editing_locks
    SET active = FALSE
    WHERE active = TRUE AND expires_at < NOW();

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_editing_locks_limpiar_vencidos
BEFORE INSERT ON editing_locks
FOR EACH ROW EXECUTE FUNCTION fn_limpiar_bloqueos_vencidos();
```
**Cómo comprobarlo:**
```sql
-- 1. Insertar un bloqueo YA vencido (a propósito)
INSERT INTO editing_locks (tenanttemplate_id, locked_by, locked_at, expires_at, active)
VALUES (1, 1, NOW() - INTERVAL '1 hour', NOW() - INTERVAL '30 minutes', TRUE);

-- 2. Insertar CUALQUIER otro bloqueo nuevo (esto dispara la limpieza del anterior)
INSERT INTO editing_locks (tenanttemplate_id, locked_by, locked_at, expires_at, active)
VALUES (2, 1, NOW(), NOW() + INTERVAL '10 minutes', TRUE);

-- 3. Verificar
SELECT * FROM editing_locks;
```
El primer bloqueo (el vencido) debe aparecer con `active = f` después del segundo `INSERT`, aunque se insertó con `active = TRUE` — quedó marcado como inactivo por la limpieza que disparó la siguiente inserción.

---

## Estado final

Las 7 secciones del examen están completas y cada pieza fue probada contra una instancia real de PostgreSQL (creación de tablas, inserción de datos, ejecución de las 68 consultas, y ejecución/disparo de los 15 procedimientos, 8 funciones y 15 triggers, incluyendo los casos que deben fallar).