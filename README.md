# Módulo: Lenguajes de Marcas y Sistemas de Gestión (0373)
### Proyecto Intermodular · 1º ASIR · Yassin Mansouri · Agencia Espanito

---

## ¿Qué datos representa el XML?

El archivo `datos.xml` representa una **exportación de solicitudes de admisión**. Contiene las candidaturas de los estudiantes aceptados o en proceso, incluyendo sus datos personales, el programa y universidad de destino, el estado del trámite de visado y el asesor responsable.

Este formato XML simula el documento que la agencia Espanito enviaría a las universidades colaboradoras como confirmación oficial de las candidaturas.

---

## Estructura del XML

```
espanito
├── agencia              → Información institucional de la agencia
└── solicitudes          → Listado de candidaturas (atributo: total)
    └── solicitud [1..*] → Cada candidatura (atributos: id, prioridad)
        ├── estudiante
        │   ├── datos personales
        │   ├── pasaporte (atributos: numero, expira)
        │   └── idiomas
        │       └── idioma [1..5] (atributo: nivel MCER)
        ├── programa
        │   └── coste_anual (atributo: moneda)
        ├── universidad
        ├── estado_solicitud
        ├── fecha_solicitud
        ├── fecha_respuesta  [opcional]
        ├── tramite_visa
        │   └── fecha_cita   [opcional]
        └── asesor
```

---

## Restricciones implementadas en el XSD

| Campo | Tipo / Restricción |
|---|---|
| `solicitud/@id` | Patrón `SOL-[0-9]{3}` |
| `solicitud/@prioridad` | Entero entre 1 y 3 |
| `pasaporte/@numero` | Patrón `[A-Z]{2}[0-9]{6}` |
| `nota_media` | Decimal entre 0.00 y 20.00 |
| `duracion_meses` | Entero entre 1 y 120 |
| `coste_anual` | Decimal positivo con 2 decimales |
| `idioma/@nivel` | ENUM: nativo, A1, A2, B1, B2, C1, C2 |
| `nivel` (programa) | ENUM: Bachillerato, Licenciatura, Maestria, Doctorado, Intercambio |
| `pais` | ENUM: España, Alemania, Francia, Paises Bajos, Reino Unido... |
| `estado_solicitud` | ENUM: Pendiente, En_proceso, Aceptada, Rechazada... |
| `estado` (visa) | ENUM: No_iniciado, Documentacion, Cita_pendiente, Aprobada... |
| `tipo` (visa) | ENUM: Estudio_larga_duracion, Schengen, Student_visa_UK... |
| `email` | Patrón regex formato estándar |
| `ranking` | Patrón `TOP \d+` |
| `version_exportacion` | Patrón `\d+\.\d+` |
| `fecha_cita`, `numero_expediente` | Opcionales (minOccurs=0) |
| `idiomas` | Entre 1 y 5 elementos idioma (minOccurs=1, maxOccurs=5) |
| `solicitud` | Mínimo 1, sin límite máximo (minOccurs=1, maxOccurs=unbounded) |

---

## Cómo se valida

### Con Python (lxml)
```bash
pip install lxml
python3 -c "
from lxml import etree
schema = etree.XMLSchema(etree.parse('esquema.xsd'))
doc = etree.parse('datos.xml')
print('VALIDO' if schema.validate(doc) else schema.error_log)
"
```

### Con xmllint (Linux/Mac)
```bash
xmllint --schema esquema.xsd datos.xml --noout
```

### Con VS Code
Instalar extensión **XML (Red Hat)** → el editor valida automáticamente al abrir `datos.xml`.

---

## Cómo encaja en el proyecto

Este módulo se integra con `espanito_db` (módulo 0372) de la siguiente forma:

```
espanito_db (MySQL)
     │
     │  SELECT solicitudes + estudiantes + programas + tramites_visa
     │
     ▼
datos.xml  →  validado por  →  esquema.xsd
     │
     │  Enviado a universidades como reporte oficial de candidaturas
     ▼
Universidad destino (TUM, UCL, Paris-Saclay...)
```

Los datos del XML son consistentes con los registros de la base de datos del proyecto: mismos estudiantes, mismas universidades, mismos estados de visa.

---

## Archivos incluidos

| Archivo | Descripción |
|---|---|
| `datos.xml` | Exportación de 5 solicitudes válidas desde espanito_db |
| `esquema.xsd` | Esquema de validación con tipos, ENUMs y restricciones |
| `datos_incorrecto.xml` | XML con 6 errores intencionales para demostrar el XSD |
| `evidencia_validacion.txt` | Log completo de ambas validaciones |
| `README.md` | Este archivo |
