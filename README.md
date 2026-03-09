# GAUSS → CKAN Importer

Script en **Python** para importar una lista de guías docentes (PDF) desde un fichero CSV y registrarlas como **datasets y recursos remotos** en un portal **CKAN** usando la **Action API**.

Este script fue diseñado para procesar el índice exportado desde **GAUSS UPM**, donde cada fila del CSV contiene información de una asignatura y la URL de su guía docente en PDF.

El programa:

1. Lee un fichero CSV.
2. Crea (o actualiza) un **dataset en CKAN** por cada fila.
3. Registra la **URL del PDF como recurso remoto** dentro del dataset.
4. Evita duplicar datasets o recursos si ya existen.

---

# Requisitos

* Python **3.8+**
* Acceso a una instancia de **CKAN**
* **API Key** de CKAN con permisos para crear datasets y recursos

Instalar dependencias:

```bash
pip install ckanapi
```

---

# Estructura del CSV

El script espera un CSV con columnas similares a las siguientes:

| Campo           | Descripción                    |
| --------------- | ------------------------------ |
| academic_year   | Año académico                  |
| semester        | Semestre                       |
| study_plan_type | Tipo de plan                   |
| study_plan_code | Código del plan                |
| study_plan_name | Nombre del plan                |
| subject_type    | Tipo de asignatura             |
| subject_code    | Código de asignatura           |
| subject_name    | Nombre de la asignatura        |
| guide_pdf_url   | URL del PDF de la guía docente |

También pueden aparecer otras URLs con información adicional:

* info_description_url
* info_professors_url
* info_prev_requirements_url
* info_competences_and_results_url
* info_syllabus_url
* info_schedule_url
* info_evaluation_url
* info_resources_url
* info_other_url

Estas se guardan como **metadata extra (`extras`)** dentro del dataset.

---

# Instalación

Clonar o copiar el script:

```bash
git clone <repo>
cd gauss-ckan-importer
```

Instalar dependencias:

```bash
pip install ckanapi
```

---

# Uso

```bash
python import_gauss_to_ckan.py \
  --csv gauss-index.csv \
  --ckan-url https://datos.midominio.es \
  --api-key TU_API_KEY \
  --owner-org mi-organizacion
```

### Parámetros

| Argumento      | Descripción                                  |
| -------------- | -------------------------------------------- |
| `--csv`        | Ruta al fichero CSV                          |
| `--ckan-url`   | URL base del portal CKAN                     |
| `--api-key`    | API token del usuario CKAN                   |
| `--owner-org`  | Organización CKAN propietaria del dataset    |
| `--license-id` | (Opcional) licencia CKAN                     |
| `--limit`      | (Opcional) número máximo de filas a procesar |

Ejemplo:

```bash
python import_gauss_to_ckan.py \
  --csv gauss-index.csv \
  --ckan-url https://datos.upm.es \
  --api-key 1234567890abcdef \
  --owner-org upm
```

---

# Funcionamiento interno

El script utiliza la **CKAN Action API** a través de la librería `ckanapi`.

Principales operaciones:

| Operación         | Acción                            |
| ----------------- | --------------------------------- |
| `package_show`    | Comprueba si el dataset ya existe |
| `package_create`  | Crea el dataset                   |
| `package_patch`   | Actualiza metadatos si ya existe  |
| `resource_create` | Añade el PDF como recurso         |

---

# Modelo de datos en CKAN

## Dataset

Se crea un dataset por asignatura y curso académico.

El nombre del dataset se genera automáticamente:

```
{academic_year}-{semester}-{study_plan_code}-{subject_code}-{subject_name}
```

Este nombre se convierte a un **slug válido para CKAN**.

Ejemplo:

```
2024-2025-1-ETSIT-12345-programacion-orientada-a-objetos
```

### Campos del dataset

| Campo CKAN | Valor                         |
| ---------- | ----------------------------- |
| name       | slug generado                 |
| title      | Nombre de asignatura + código |
| notes      | descripción generada          |
| owner_org  | organización CKAN             |
| tags       | año, semestre, códigos        |
| extras     | metadatos del CSV             |

---

## Recurso

Cada dataset contiene un recurso:

```
Guía docente PDF
```

Propiedades del recurso:

| Campo         | Valor           |
| ------------- | --------------- |
| url           | guide_pdf_url   |
| format        | PDF             |
| mimetype      | application/pdf |
| resource_type | file            |

El fichero **no se sube a CKAN**, se registra como **URL remota**.

---

# Prevención de duplicados

El script implementa dos mecanismos de seguridad:

### Dataset duplicado

Antes de crear un dataset se ejecuta:

```
package_show
```

Si ya existe:

```
package_patch
```

para actualizarlo.

---

### Recurso duplicado

Antes de crear el recurso se revisan los recursos existentes del dataset:

```
resource.url == guide_pdf_url
```

Si ya existe, se omite.

---

# Ejecución parcial (testing)

Para probar el script sin importar todo el CSV:

```bash
python import_gauss_to_ckan.py \
  --csv gauss-index.csv \
  --ckan-url https://datos.upm.es \
  --api-key XXXXX \
  --owner-org upm \
  --limit 10
```

---

# Ejemplo de salida

```
[OK] Dataset creado: 2024-2025-1-ETSIT-12345-programacion
[OK] Recurso PDF creado en dataset

[OK] Dataset actualizado: 2024-2025-1-ETSIT-12346-algoritmos
[SKIP] Recurso ya existe
```

---

# Posibles mejoras

Algunas extensiones que se pueden añadir fácilmente:

* crear **un único dataset por plan de estudios**
* añadir **todos los enlaces GAUSS como recursos**
* descargar los PDFs y **subirlos a CKAN**
* mapear metadatos a **DCAT / RDF**
* paralelizar el proceso
* añadir **logs estructurados**

---

# Autor

Script desarrollado para automatizar la publicación de **guías docentes GAUSS en CKAN**.
