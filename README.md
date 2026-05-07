[![DianaLabs - Koha Actualizados](https://img.shields.io/badge/DianaLabs-Koha_Actualizados-8A2BE2?logo=github)](https://github.com/dianahuarazvargas-bit/dianalabs-koha-Actualizados)

[![Hecho con Colab](https://img.shields.io/badge/Hecho_con-Google_Colab-F9AB00?logo=googlecolab)](https://colab.research.google.com/)
[![MARC21](https://img.shields.io/badge/Formato-MARC21-blue)](https://www.loc.gov/marc/)
[![Koha](https://img.shields.io/badge/Integración-Koha-0066CC?logo=koha)](https://koha-community.org/)

## 📘 Guía de Usuario : Actualización masiva de registros existentes en Koha (MARC21)

Esta guía explica cómo **actualizar registros bibliográficos ya existentes** en Koha usando un flujo automatizado que combina:
- El archivo global de la colección (exportado desde Koha)
- Una hoja de actualizaciones (con los cambios que quieres aplicar)

---

## 📁 Archivos del repositorio

| Archivo | Descripción |
|---------|-------------|
| `GoogleSheet_a_Marc_Libros_Actualizados.ipynb` | Notebook de Google Colab que procesa y actualiza registros MARC21 |
| `README.md` | Esta guía — instrucciones completas |

---

## 🧠 Lógica del proceso (4 etapas)

---

## 🔧 1. Preparación de los datos

### Archivo original (desde Koha)
Exporta todos los registros de tu colección desde Koha como hoja de cálculo.  
Este archivo se llamará `Normalized_Global_Report` y debe tener **las mismas columnas** que se muestran en el modelo de este repositorio ya que se genera con la query en la sección de informes guardados de KOHA.

### Hoja de actualizaciones
Crea un Google Sheets con **solo los registros que quieres modificar**.  
Debe contener:
- La columna `biblionumber_999_c` (identificador único en Koha)
- Las columnas con los **datos actualizados** (ej. `cd_dewey_082`, `autor_institucion_110`, etc.). Se adjunta modelo de este documento en los archivos del presente repositorio.

---

## 🐍 2. Explicación del script paso a paso

### Paso 1: Cargar el libro original
Lee el archivo `Normalized_Global_Report` (todos los registros existentes en la colección).

### Paso 2: Cargar la hoja de actualizaciones
Solo para **visualizar su contenido** y comprobar que es el listado correcto. No se modifica nada aún.

### Paso 3: Normalizar columnas
Asegura que ambos archivos tengan la misma estructura:
- Elimina la columna `biblionumber` (duplicada)
- Renombra columnas para que coincidan (`coau_institucion_1` → `coau_institución_1`, `branch_952_a` → `952_a`)
- Añade columnas faltantes: `numero_copia_952_t`, `952_b`, `numero_copia_952_t.1`, `numero_copia_952_t.2`

### Paso 4: Limpieza e integración de datos
Este paso es el **corazón del proceso**:

| Acción | Qué hace |
|--------|----------|
| **Filtrar** | Solo mantiene los registros del original que están en la hoja de actualizaciones |
| **Alertar** | Si un `biblionumber` en actualizaciones no existe en el original → te avisa |
| **Colapsar** | Si un libro tiene múltiples ejemplares, los agrupa en **una sola fila** con columnas `Ej. 2`, `Ej. 3` |
| **Alertar por exceso** | Si un libro tiene más de 3 ejemplares, te avisa (solo procesa hasta `Ej. 3`) |
| **Sobrescribir** | Los datos de la hoja de actualizaciones **reemplazan** a los del original |

### Paso 5: Conversión a MARC21
Genera un archivo `.mrc` con todos los campos MARC estándar:
- `020` ISBN, `022` ISSN, `024` DL
- `040` Fuente, `041` Idioma
- `082` Dewey
- `100`/`110`/`111` Autores
- `245` Título, `250` Edición, `260` Publicación, `300` Descripción
- `490` Series, `500` Notas, `505` Contenido, `520` Resumen
- `600`/`610`/`650`/`651` Materias
- `700`/`710` Coautores
- `942` Tipo de ítem (Koha)
- `952` Ejemplares (con soporte para `Ej. 2`, `Ej. 3`)
- `999` Biblionumber (Koha)

### Paso 5.1: Vista previa
Muestra los primeros 5 registros en formato mnemotécnico para que verifiques antes de descargar.

### Paso 6: Descarga
Genera y descarga automáticamente el archivo:


---

## ⚙️ 3. Ejecución en Google Colab

1. **Abre el notebook** en Google Colab  
   `GoogleSheet_a_Marc_Libros_Actualizados.ipynb`

2. **Actualiza las URLs** dentro del código:
   - `google_sheet_original_url` → enlace al archivo global de tu colección
   - `google_sheet_updated_url` → enlace a la hoja de actualizaciones

3. **Ejecuta todas las celdas**  
   Ve a: `Entorno de ejecución > Ejecutar todas`

4. **Revisa las alertas**  
   El script te avisará si:
   - Hay `biblionumber` en actualizaciones que no existen en el original
   - Hay libros con más de 3 ejemplares

5. **Visualiza la vista previa** del archivo MARC

6. **Descarga el archivo** `.mrc` generado

---

## 📥 4. Importación en Koha

Una vez descargado el archivo `.mrc`:

1. Flujo:  
   `Herramientas > Subir archivos MARC para importación > btn: Seleccionar archivo > Regla de coincidencia de registro: Busca registros coincidentes > Acción en caso de encontrar registro coincidente: Reemplace un registro existente con uno nuevo > btn: Seleccione el archivo a importar > btn: Administrar registros preparados > btn: importar este lote en el catálogo`

2. **Selecciona el archivo** `koha_Libros_Actualizados_from_GoogleSheet.mrc`

3. **El sistema procesa el archivo** para revisión

4. **El sistema verifica que los `biblionumber` coincidan** con los registros existentes

5. **Completa la importación** — Koha actualizará automáticamente los campos modificados

---
## ⚠️ Especificaciones técnicas (diseñadas a propósito)

Las siguientes características son **comportamientos intencionales** del script, no errores:

| Característica | Comportamiento | Razón |
|----------------|----------------|-------|
| **Requiere `biblionumber`** | La hoja de actualizaciones debe tener esta columna obligatoriamente | Es el identificador único que vincula los registros en Koha |
| **No sobrescribe campos vacíos** | Los campos que dejes vacíos en la hoja de actualizaciones **no se borran** en Koha | El script solo actualiza lo que explícitamente pones; los datos existentes se conservan |

> ✅ **Estos comportamientos son correctos y no serán modificados.**

---

## 🚫 Limitaciones técnicas conocidas

Las siguientes son **limitaciones reales** del script que podrían mejorarse en versiones futuras:

### 1. Máximo 3 ejemplares por registro
| Situación | Comportamiento |
|-----------|----------------|
| Libro con 1 ejemplar | ✅ Procesa normalmente |
| Libro con 2-3 ejemplares | ✅ Procesa normalmente |
| Libro con más de 3 ejemplares | ⚠️ El script **alerta** pero solo procesa hasta `Ej. 3` |

### 2. Actualización de clasificaciones por ejemplar
**¿Qué funciona?** ✅ Actualizar campos del registro bibliográfico (título, autor, ISBN, etc.)

**¿Qué NO funciona?** ❌ Actualizar clasificaciones individuales (`952$o` o `dewey_952_o`) cuando un libro tiene **múltiples ejemplares con valores distintos**

**🔴 Ejemplo del problema:**

| Situación original en Koha | Intentas actualizar a | Resultado |
|---------------------------|----------------------|-----------|
| Ej. 1: `320.985/P43/2020` | `330.985/N44/2023` | ❌ No se actualiza |
| Ej. 2: `320.985/P43/2021` | `330.985/N44/2023` | ❌ No se actualiza |
| Ej. 3: `320.985/P43/2022` | `330.985/N44/2023` | ❌ No se actualiza |

Los demás campos (título, autor, etc.) **sí se actualizan correctamente**, pero la clasificación de los ejemplares no.

**🛠️ Soluciones posibles:**

| Opción | Qué hacer | Dificultad |
|--------|-----------|------------|
| **Manual** | Actualizar clasificaciones directamente en Koha, uno por uno | Baja |
| **Reemplazar** | Eliminar los ejemplares y volver a crearlos con la nueva clasificación | Media |
| **Modificar el script** | Añadir columnas `dewey_952_o.1`, `dewey_952_o.2`, `dewey_952_o.3` | Alta (requiere desarrollo) |

> 📌 **Recomendación:** Para actualizar clasificaciones en libros con múltiples ejemplares, hazlo manualmente en Koha. El script no está diseñado para este caso específico.

---

## 📊 Resumen rápido

| Tipo | Característica | Estado |
|------|----------------|--------|
| ✅ Diseñado | Requiere `biblionumber` | Intencional |
| ✅ Diseñado | No sobrescribe campos vacíos | Intencional |
| ⚠️ Limitación | Máximo 3 ejemplares | Mejorable |
| ⚠️ Limitación | Actualización de clasificaciones múltiples | Mejorable |
