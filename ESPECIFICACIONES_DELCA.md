# REENCAUCHADORA DELCA SAS — Especificaciones Técnicas del Sistema
## Documento para Claude Code · Versión 1.0

---

## 1. DESCRIPCIÓN GENERAL

Sistema de gestión operativa y administrativa de escritorio para Windows. Reemplaza el sistema actual (Visual FoxPro/Access) centralizando el control de llantas, clientes, producción y finanzas en una sola plataforma moderna, robusta y visualmente amigable.

**Usuarios:** 2-3 simultáneos en red local  
**Sistema operativo:** Windows 10/11  
**Resolución mínima:** 1366x768  

---

## 2. STACK TECNOLÓGICO

| Componente | Tecnología | Justificación |
|---|---|---|
| Lenguaje | Python 3.11+ | Estable, amplio ecosistema |
| Interfaz | CustomTkinter 5.x | Moderno, nativo Windows, tema claro/oscuro |
| Base de datos | SQLite 3 + SQLAlchemy 2.x | Local, sin servidor, robusto |
| Impresión | ReportLab 4.x | Posicionamiento exacto sobre formulario preimpreso |
| Íconos | Lucide o FontAwesome TTF | Consistencia visual |
| Empaquetado | PyInstaller | Genera .exe sin dependencias externas |
| Respaldos | Módulo shutil + schedule | Copia automática diaria de la BD |

---

## 3. ESTRUCTURA DE CARPETAS DEL PROYECTO

```
delca_sistema/
├── main.py                    # Punto de entrada
├── config.py                  # Configuración global (colores, fuentes, rutas)
├── requirements.txt
├── assets/
│   ├── logo_delca.png
│   ├── fonts/
│   └── icons/
├── database/
│   ├── db.py                  # Conexión SQLAlchemy
│   ├── models.py              # Modelos ORM
│   ├── migrations.py          # Creación inicial de tablas
│   └── backup.py              # Respaldo automático
├── modules/
│   ├── ingreso/
│   │   ├── clientes.py        # Formulario clientes
│   │   └── llantas.py         # Formulario llantas + vista previa impresión
│   ├── principal/
│   │   ├── main_view.py       # Vista principal
│   │   ├── tabla_llantas.py   # Tabla central con semáforo
│   │   └── acciones.py        # Modales de acciones rápidas
│   ├── reportes/
│   │   ├── reportes_view.py   # Vista contenedor
│   │   └── generadores.py     # Lógica de cada reporte
│   ├── metricas/
│   │   ├── metricas_view.py   # Dashboard KPIs
│   │   ├── graficas.py        # Gráficas de barras y distribución
│   │   ├── simulador.py       # Simulador materia prima
│   │   └── precios.py         # Tabla de precios
│   └── impresion/
│       ├── orden_reencauche.py # Impresión REE-F-010
│       └── calibracion.py      # Configuración de coordenadas
├── utils/
│   ├── validaciones.py        # Validadores de campos
│   ├── exportar.py            # Exportar a Excel/CSV
│   └── logger.py              # Log de cambios
└── data/
    ├── delca.db               # Base de datos principal
    ├── backups/               # Respaldos automáticos
    └── logs/                  # Registro de operaciones
```

---

## 4. DISEÑO UI — GUÍA DE ESTILOS

### Paleta de colores
```python
COLORS = {
    # Primarios
    "primary":        "#534AB7",   # Morado Delca — botones principales, tabs activos
    "primary_hover":  "#3C3489",   # Hover botones primarios
    "primary_light":  "#EEEDFE",   # Fondos seleccionados, badges activos

    # Semáforo llantas
    "sem_green":      "#639922",   # < 3 meses en planta
    "sem_yellow":     "#BA7517",   # 3-12 meses
    "sem_red":        "#A32D2D",   # > 12 meses

    # Estados
    "estado_pend_bg": "#FAEEDA",   # Pendiente fondo
    "estado_pend_tx": "#854F0B",   # Pendiente texto
    "estado_reenc_bg":"#EAF3DE",   # Reencauchada fondo
    "estado_reenc_tx":"#3B6D11",   # Reencauchada texto
    "estado_insp_bg": "#E6F1FB",   # Inspección fondo
    "estado_insp_tx": "#185FA5",   # Inspección texto
    "estado_rech_bg": "#FCEBEB",   # Rechazada fondo
    "estado_rech_tx": "#A32D2D",   # Rechazada texto

    # Financiero
    "fin_ok":         "#0F6E56",   # Pagado / positivo
    "fin_danger":     "#A32D2D",   # Saldo / negativo
    "fin_warn":       "#854F0B",   # Advertencia

    # Neutros
    "bg_primary":     "#FFFFFF",
    "bg_secondary":   "#F5F4F2",
    "bg_tertiary":    "#EEECE8",
    "border":         "#E0DDD8",
    "text_primary":   "#1A1916",
    "text_secondary": "#6B6860",
    "text_tertiary":  "#9B9890",
}
```

### Tipografía
```python
FONTS = {
    "title":    ("Segoe UI", 15, "bold"),       # Títulos de ventana
    "subtitle": ("Segoe UI", 13, "bold"),       # Títulos de sección
    "body":     ("Segoe UI", 12),               # Texto general
    "small":    ("Segoe UI", 11),               # Labels secundarios
    "tiny":     ("Segoe UI", 10),               # Etiquetas de campos
    "mono":     ("Consolas", 12),               # Tiquetes, números
    "kpi":      ("Segoe UI", 24, "bold"),       # Números KPI grandes
}
```

### Principios de diseño UI
- **Espaciado generoso:** padding mínimo 12px en cards, 8px entre elementos
- **Bordes sutiles:** 1px, color `#E0DDD8`, radio 8px en cards, 6px en inputs
- **Sombras suaves:** cards con `relief="flat"` + borde, sin sombras pesadas
- **Estados visuales claros:** hover, selected, disabled bien diferenciados
- **Íconos acompañando texto** en todos los botones principales
- **Feedback inmediato:** mensajes de éxito/error en la misma vista, no popups molestos
- **Confirmaciones solo en acciones destructivas:** eliminar, cambiar tiquete, reasignar cliente

---

## 5. BASE DE DATOS — MODELOS

### Tabla: clientes
```sql
CREATE TABLE clientes (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre      TEXT NOT NULL,
    nit_cc      TEXT UNIQUE NOT NULL,
    telefono    TEXT,
    direccion   TEXT,
    correo      TEXT,
    asesor      TEXT,
    tipo        TEXT DEFAULT 'P',  -- P=Particular, E=Empresa
    activo      INTEGER DEFAULT 1,
    creado_en   TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    modificado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla: llantas
```sql
CREATE TABLE llantas (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    tiquete         TEXT UNIQUE NOT NULL,
    orden           TEXT NOT NULL,        -- N° Orden de Servicio
    cliente_id      INTEGER NOT NULL REFERENCES clientes(id),
    marca           TEXT,
    dimension       TEXT,
    disenio         TEXT,                 -- Diseño/Banda
    dot_serie       TEXT,
    ubicacion       TEXT DEFAULT 'P',
    estado          TEXT DEFAULT 'PENDIENTE',
    -- Estados: PENDIENTE, INSPECCION_INICIAL, EN_PROCESO,
    --          INSPECCION_FINAL, REENCAUCHADA, RECHAZADA, SOPLADA, REPARADA
    resolucion_inspeccion TEXT,
    prioridad       INTEGER DEFAULT 2,
    valor_servicio  REAL,
    observacion     TEXT,
    fecha_entrada   DATE NOT NULL,
    fecha_salida    DATE,
    mes_procesado   INTEGER,
    anio_procesado  INTEGER,
    costo_kg        REAL,
    doc_salida      TEXT,
    creado_en       TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    modificado_en   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla: pagos
```sql
CREATE TABLE pagos (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    cliente_id  INTEGER NOT NULL REFERENCES clientes(id),
    valor       REAL NOT NULL,
    fecha       DATE NOT NULL,
    observacion TEXT,
    usuario     TEXT,
    creado_en   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla: precios
```sql
CREATE TABLE precios (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    dimension   TEXT NOT NULL,
    disenio     TEXT NOT NULL,
    precio      REAL NOT NULL,
    UNIQUE(dimension, disenio),
    modificado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla: metas
```sql
CREATE TABLE metas (
    id                  INTEGER PRIMARY KEY AUTOINCREMENT,
    meta_llantas_mes    INTEGER DEFAULT 60,
    punto_equilibrio    INTEGER DEFAULT 40,
    meta_ingresos_mes   REAL    DEFAULT 25000000,
    valor_promedio_llanta REAL  DEFAULT 414000,
    modificado_en       TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla: historial_cambios (auditoría)
```sql
CREATE TABLE historial_cambios (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    tabla       TEXT NOT NULL,
    registro_id INTEGER NOT NULL,
    campo       TEXT NOT NULL,
    valor_antes TEXT,
    valor_despues TEXT,
    usuario     TEXT,
    fecha       TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla: materia_prima (pesos para simulador)
```sql
CREATE TABLE materia_prima (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    dimension       TEXT NOT NULL,
    disenio         TEXT NOT NULL,
    peso_banda_kg   REAL NOT NULL,
    peso_cushion_kg REAL NOT NULL,
    UNIQUE(dimension, disenio)
);
```

---

## 6. MÓDULOS — ESPECIFICACIÓN DETALLADA

### 6.1 TOPBAR (componente global)
- Logo Delca izquierda
- Buscador central con selector de tipo: nombre / cédula-NIT / tiquete / orden / dimensión
- 4 botones de navegación: Principal · Ingreso · Reportes · Métricas
- Estado activo resaltado en morado

### 6.2 MÓDULO INGRESO

#### Formulario Clientes
**Campos:**
- Nombre completo (obligatorio)
- NIT/Cédula (obligatorio, único)
- Teléfono/Celular (obligatorio)
- Asesor (select: COM, VEN, DIR)
- Dirección
- Tipo (P-Particular / E-Empresa)
- Correo (opcional)

**Comportamiento:**
- Buscador superior: al escribir 3+ caracteres muestra cliente si existe y carga sus datos
- Si existe: modo EDICIÓN (modifica datos)
- Si no existe: modo CREACIÓN (nuevo registro)
- Validación en tiempo real: NIT no puede duplicarse
- Botón Guardar deshabilitado hasta que campos obligatorios estén llenos

#### Formulario Llantas
**Campos obligatorios:** N° Tiquete, N° Orden, Cliente (vinculado), Fecha entrada, Marca, Dimensión, Diseño, Ubicación inicial  
**Campos opcionales:** DOT/Serie, Valor servicio, Prioridad, Observación

**Comportamiento especial:**
- Al seleccionar Dimensión + Diseño: busca precio en tabla `precios` y lo autocompleta en "Valor servicio"
- Vista previa de la orden en tiempo real (lado derecho del formulario)
- Botón "Guardar e imprimir": guarda en BD y abre diálogo de impresión
- Botón "Solo guardar": guarda sin imprimir
- Validación: tiquete único (no puede existir ya en BD)
- N° Orden: puede repetirse (una orden tiene varios tiquetes)

### 6.3 MÓDULO PRINCIPAL

#### Búsqueda global
- Select tipo + input texto
- Busca en tiempo real (desde 2 caracteres)
- Resultado: carga el cliente encontrado en panel izquierdo y sus llantas en tabla central
- Si busca por tiquete/orden: va directo a la llanta y selecciona la fila

#### Panel izquierdo — Ficha cliente
- Nombre, badge estado pago, NIT/CC, teléfono, dirección, asesor
- Resumen financiero: total llantas activas, total servicio, total abonado, saldo
- Filtros estado: Todas / Pendiente / Inspección / Reencauchada / Rechazada
- Filtros pago: Todas / Pagadas / Pendientes

#### Tabla central — Llantas
Columnas: semáforo | tiquete | orden | tiempo | marca | dimensión | diseño | estado | ubicación | pago

**Semáforo de tiempo:**
- Verde: fecha_entrada hace menos de 90 días
- Amarillo: 90-365 días
- Rojo: más de 365 días

**Selección:** clic en fila la marca como seleccionada (fondo morado suave)

#### Acciones rápidas (menú desplegable)
Actúan sobre la llanta seleccionada. Cada acción abre un modal:

1. **Cambiar estado** — select con todos los estados posibles + observación opcional
2. **Cambiar ubicación** — select con ubicaciones + fecha cambio
3. **Registrar abono** — valor, fecha, observación. Muestra saldo actual del cliente
4. **Registrar salida** — estado al salir, fecha, destino
5. **Asignar valor llanta** — valor manual o desde tabla de precios
6. **Editar datos llanta** — formulario completo editable: cliente (reasignar), tiquete, orden, marca, dimensión, diseño, DOT, ubicación, valor, observación

### 6.4 MÓDULO REPORTES

Menú lateral izquierdo. Cada reporte tiene filtros en la parte superior, tabla de resultados y botones Exportar / Imprimir al final.

| Reporte | Filtros | Columnas principales |
|---|---|---|
| Llantas en planta | cliente, orden, dimensión, marca, estado, agrupar | semáforo, tiquete, orden, cliente, tel, marca, dim, diseño, estado, ubic, tiempo |
| Por cliente | nombre o NIT | tiquete, orden, marca, dim, diseño, estado, ubic, tiempo |
| Por marca | marca, estado | marca, cantidad, %, pendientes, reencauchadas |
| Por dimensión | dimensión | dimensión, en planta, %, pendientes, reencauchadas |
| Por diseño/banda | diseño | diseño, en planta, %, dimensión más común |
| Reencauches mes | mes, año, tipo | dimensión, diseño, cantidad, % mes, valor |
| Reencauches año | año | mes, llantas, % meta, ingresos |
| Producción general | año, dimensión, marca | totales, tasa rechazo, días promedio, barras participación |

**Encabezado de todos los reportes imprimibles:**
```
REENCAUCHADORA DELCA SAS
NIT: [NIT empresa]
[Fecha de impresión] — [Nombre del reporte]
```

### 6.5 MÓDULO MÉTRICAS

#### KPIs del mes
- 4 tarjetas: llantas producidas, cumplimiento %, ingresos, tasa rechazo
- 3 barras de progreso: llantas vs meta, ingresos vs meta, punto de equilibrio
- 2 cards: semáforo cantidades en planta, comparativo vs mes anterior
- Selector de mes/año en topbar del módulo

#### Producción
- Gráfica de barras: últimos 6 meses (verde≥meta, amarillo cerca, rojo bajo)
- 2 distribuciones: por dimensión y por banda (barras horizontales con %)

#### Financiero
- Grid 6 meses: mes, llantas, ingresos
- Resumen acumulado anual: 4 KPIs

#### Simulador materia prima
- Sliders KG banda por referencia (VZY2, PBA60, REP, ST + las que se agreguen)
- Sliders KG cushion por referencia
- Resultado automático: llantas posibles por referencia + total
- Pesos tomados de tabla `materia_prima` en BD (editables)

#### Tabla de precios
- CRUD completo: agregar, editar, eliminar combinaciones dimensión+diseño
- Campo precio editable inline
- Botón "Guardar todos"
- Nota: al ingresar llanta con esa combinación, el precio se autocompleta

#### Configurar metas
- Meta llantas/mes, punto equilibrio, meta ingresos/mes, valor promedio llanta
- Vista previa de KPIs con los nuevos valores antes de guardar

### 6.6 MÓDULO IMPRESIÓN — REE-F-010

El formulario físico tiene medidas: **media hoja carta (21.6cm x 14cm), orientación horizontal.**

**Campos a imprimir y sus coordenadas (en mm desde esquina superior izquierda):**

| Campo | X (mm) | Y (mm) | Tam. fuente |
|---|---|---|---|
| **TIQUETE SUPERIOR (desprendible)** | | | |
| Cliente top | 5 | 8 | 9pt |
| Tiquete top | 5 | 20 | 11pt bold |
| Dimensión top | 35 | 20 | 9pt |
| Diseño top | 75 | 20 | 9pt |
| O.S. top | 5 | 30 | 9pt |
| Serie top | 35 | 30 | 9pt |
| **HOJA DE PROCESO (parte inferior)** | | | |
| Cliente | 5 | 52 | 9pt |
| Tiquete | 5 | 63 | 11pt bold |
| Dimensión | 35 | 63 | 9pt |
| Marca | 75 | 63 | 9pt |
| Diseño | 5 | 74 | 9pt |
| Serie | 35 | 74 | 9pt |
| Fecha | 75 | 74 | 9pt |
| O.S. | 5 | 85 | 9pt |
| Observación | 35 | 85 | 8pt |

> **NOTA CRÍTICA:** Estas coordenadas son un punto de partida. Se debe implementar una pantalla de calibración donde el usuario puede ajustar cada valor con flechas +/- de 0.5mm y guardar la configuración. La calibración final se hace imprimiendo sobre una hoja en blanco, comparando con el formulario físico y ajustando hasta lograr precisión exacta.

**Implementación:**
```python
# Usar ReportLab con canvas
from reportlab.pdfgen import canvas
from reportlab.lib.units import mm

def imprimir_orden(datos, config_coords):
    c = canvas.Canvas("orden_temp.pdf", pagesize=(216*mm, 140*mm))
    c.setFont("Helvetica-Bold", 11)
    c.drawString(config_coords['tiquete_x']*mm, 
                 config_coords['tiquete_y']*mm, 
                 datos['tiquete'])
    # ... resto de campos
    c.save()
    # Enviar a impresora por defecto de Windows
    import subprocess
    subprocess.run(['start', '/wait', '', 'orden_temp.pdf'], shell=True)
```

---

## 7. ROBUSTEZ Y SEGURIDAD

### Validaciones obligatorias
- Tiquete: único, solo alfanumérico
- NIT/CC: único por cliente
- Fecha: no puede ser futura para entradas ya registradas
- Valor: solo números positivos
- Campos obligatorios: UI bloquea el guardado y resalta en rojo

### Manejo de errores
```python
# Patrón para todas las operaciones de BD
def guardar_llanta(datos):
    try:
        with db.session.begin():
            # operación
            db.session.add(llanta)
        mostrar_exito("Llanta guardada correctamente")
    except IntegrityError as e:
        mostrar_error("El tiquete ya existe en el sistema")
    except Exception as e:
        mostrar_error("Error inesperado. Los datos no fueron modificados.")
        logger.error(f"Error guardar_llanta: {e}")
```

### Respaldo automático
```python
# Ejecutar al iniciar la app cada día
def backup_diario():
    origen = "data/delca.db"
    fecha = datetime.now().strftime("%Y%m%d")
    destino = f"data/backups/delca_{fecha}.db"
    if not os.path.exists(destino):
        shutil.copy2(origen, destino)
    # Mantener solo los últimos 30 respaldos
    limpiar_backups_antiguos(mantener=30)
```

### Auditoría de cambios
- Cada modificación a llantas o clientes registra en `historial_cambios`: tabla, id, campo, valor antes, valor después, usuario, timestamp
- Accesible desde menú Herramientas > Historial de cambios

---

## 8. MIGRACIÓN DE DATOS (FASE FINAL)

**Ejecutar SOLO cuando el sistema esté completamente operativo y probado.**

### Paso 1 — Exportar del sistema actual
- Exportar clientes a Excel/CSV
- Exportar llantas/tiquetes a Excel/CSV
- Verificar que todos los campos requeridos estén presentes

### Paso 2 — Script de importación
```python
def migrar_desde_excel(archivo_clientes, archivo_llantas):
    # Leer Excel con pandas
    # Validar cada fila antes de insertar
    # Registrar errores en archivo de log
    # Mostrar resumen: X registros importados, Y con errores
    pass
```

### Paso 3 — Verificación
- Comparar conteos: clientes, llantas totales, llantas por estado
- Verificar 10-20 registros manualmente al azar
- Solo proceder si todo cuadra

### Paso 4 — Transición
- Día de corte: último día usando sistema viejo
- Día siguiente: sistema nuevo en producción
- Sistema viejo disponible en modo solo-lectura por 30 días como respaldo

---

## 9. INSTRUCCIONES PARA CLAUDE CODE

### Orden de construcción recomendada

1. **Base de datos** — `database/models.py` + `database/migrations.py` + datos de prueba
2. **Config global** — `config.py` con colores, fuentes, constantes
3. **Ventana principal** — `main.py` + topbar + navegación entre módulos
4. **Módulo Ingreso** — clientes primero, luego llantas con vista previa
5. **Módulo Principal** — tabla con semáforo + panel cliente + acciones
6. **Módulo Reportes** — vista contenedor + los 8 reportes
7. **Módulo Métricas** — KPIs + gráficas + simulador + precios + metas
8. **Impresión** — calibración + generación PDF + envío a impresora
9. **Robustez** — validaciones, manejo de errores, backups, auditoría
10. **Empaquetado** — PyInstaller para generar .exe instalable

### Datos de prueba incluir
- 10 clientes de ejemplo
- 30-40 llantas en diferentes estados y fechas (para que el semáforo muestre los 3 colores)
- 5 meses de historial para métricas
- Precios para las 4 combinaciones principales
- Metas configuradas

### Consideraciones de diseño UI en CustomTkinter
- Usar `CTkScrollableFrame` para tablas largas
- Usar `CTkTabview` para módulo Ingreso (Clientes / Llantas)
- Usar `CTkToplevel` para modales de acciones (no `messagebox`)
- Implementar efecto hover en filas de tabla con `bind("<Enter>")` y `bind("<Leave>")`
- Separadores visuales entre secciones con `CTkFrame` de altura 1px
- Animación suave en cambio de módulos (fade o slide)
- Todos los inputs con placeholder_text descriptivo
- Labels de campos en mayúsculas pequeñas (10pt, color secundario)

---

## 10. COMANDO DE INICIO PARA CLAUDE CODE

Pegar este mensaje al iniciar la sesión en Claude Code:

```
Construye el sistema de gestión de escritorio para la Reencauchadora Delca SAS 
según el documento ESPECIFICACIONES_DELCA.md adjunto.

Empieza por:
1. Crear la estructura de carpetas completa
2. Instalar dependencias (customtkinter, sqlalchemy, reportlab, pandas, pillow)
3. Construir la base de datos con todos los modelos e insertar datos de prueba
4. Construir config.py con la paleta de colores y tipografías definidas
5. Construir main.py con la ventana principal y navegación entre los 4 módulos

En el diseño UI prioriza: espaciado generoso, tipografía Segoe UI, 
colores de la guía de estilos, íconos en botones, feedback visual inmediato.
Construye módulo por módulo y ejecuta para verificar antes de continuar.
```
