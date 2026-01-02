# Global Canopy Height

## Objetivo del Proyecto

Este proyecto permite descargar y procesar datos del **Meta/WRI Global Canopy Height Map (CHM)**, un mapa global de altura de dosel forestal de alta resolución. El objetivo es:

- Descargar tiles (teselas) de altura de dosel para una región de interés específica
- Procesar datos en formato Cloud Optimized GeoTIFF desde AWS S3
- Visualizar y analizar la altura del dosel arbóreo usando QGIS
- Integrar datos de altura de vegetación en análisis geoespaciales

Los datos provienen de imágenes satelitales procesadas por Meta y World Resources Institute, con resoluciones que varían según la región.

## Prerrequisitos

### 1. Instalar AWS CLI

El AWS CLI es necesario para descargar los datos desde el bucket público de S3. **No se requieren credenciales de AWS** ya que el bucket es público.

#### macOS

```bash
# Usando Homebrew (recomendado)
brew install awscli

# Verificar instalación
aws --version
```

#### Linux

```bash
# Descargar e instalar
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verificar instalación
aws --version
```

#### Windows

1. Descargar el instalador MSI desde: https://awscli.amazonaws.com/AWSCLIV2.msi
2. Ejecutar el instalador
3. Abrir una nueva terminal PowerShell y verificar:
```powershell
aws --version
```

**Documentación oficial:** https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

### 2. Instalar uv (Gestor de paquetes Python)

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Verificar instalación
uv --version
```

## Instalación del Entorno

1. **Clonar o navegar al directorio del proyecto:**
```bash
git clone https://github.com/Joaquin-Urruti/Global-Canopy-Height.git
cd Global-Canopy-Height
``

2. **Instalar dependencias con uv:**
```bash
# Crear entorno virtual e instalar todas las dependencias
uv sync
```

Esto instalará automáticamente:
- pandas (manipulación de datos)
- geopandas (operaciones GIS)
- jupyter/notebook (para ejecutar notebooks)

## Uso

### 1. Ejecutar Jupyter Notebook

```bash
# Iniciar Jupyter con el entorno del proyecto
uv run jupyter notebook
```

Esto abrirá Jupyter en tu navegador. Abre el archivo `download_s3_aws_images.ipynb`.

### 2. Workflow básico

El notebook incluye los siguientes pasos:

1. **Cargar índice de tiles global:**
   ```python
   tiles = gpd.read_file('tiles.geojson')
   ```

2. **Definir región de interés:**
   ```python
   roi = gpd.read_file('roi.gpkg')
   ```

3. **Identificar tiles necesarios:**
   ```python
   selected_tiles = tiles.overlay(roi, how='intersection')
   ```

4. **Descargar tiles desde S3:**
   Los comandos AWS CLI se ejecutan automáticamente en el notebook.

### 3. Visualizar en QGIS

1. Abrir `Global Canopy Height.qgz` en QGIS
2. Agregar las capas raster descargadas desde `./outputs/`
3. Los archivos `.tif` contienen la altura del dosel en metros

## Estructura del Proyecto

```
.
├── README.md                           # Este archivo
├── pyproject.toml                      # Configuración de dependencias
├── download_s3_aws_images.ipynb        # Notebook principal
├── roi.gpkg                            # Región de interés (Argentina)
├── tiles.geojson                       # Índice global de tiles (~15 MB)
├── chm_contents.txt                    # Listado completo de ~56k tiles disponibles
├── Global Canopy Height.qgz            # Proyecto QGIS
├── outputs/                            # Tiles descargados (.tif)
└── global_canopy_heght_argentina.gpkg  # Datos procesados
```

## Datos

### Fuente de Datos

- **Bucket S3:** `s3://dataforgood-fb-data/forests/v1/alsgedi_global_v6_float/`
- **Acceso:** Público (sin credenciales)
- **CRS:** EPSG:3857 (Web Mercator)
- **Formato:** Cloud Optimized GeoTIFF

### Productos Disponibles

- **CHM GeoTIFFs:** Altura del dosel (Float32, UInt16, o Byte)
- **Máscaras de nubes:** Archivos `.tif.msk` con píxeles válidos (generados bajo demanda)
- **Metadata:** Archivos `.geojson` con fechas de observación

### Escala del Dataset

El dataset global contiene aproximadamente **56,000 tiles** que cubren áreas forestales del mundo entero. La distribución de tamaños de archivo indica la densidad de cobertura forestal:

- **~8,450 tiles pequeños (~21 MB):** Áreas con poca o nula cobertura forestal
- **~15,400 tiles medianos (100-500 MB):** Áreas con cobertura forestal moderada
- **~14,400 tiles grandes (>500 MB):** Áreas con densa cobertura forestal

El archivo más grande supera los 900 MB, correspondiente a zonas de bosques muy densos. El archivo `chm_contents.txt` contiene un listado completo de todos los tiles disponibles (actualizado en abril 2024).

## Comandos Útiles

```bash
# Ver contenido del bucket S3
aws s3 ls --no-sign-request s3://dataforgood-fb-data/forests/v1/alsgedi_global_v6_float/

# Descargar un tile específico
aws s3 cp --no-sign-request \
  s3://dataforgood-fb-data/forests/v1/alsgedi_global_v6_float/chm/[QuadKey].tif \
  ./outputs/[QuadKey].tif

# Añadir nueva dependencia Python
uv add <paquete>

# Actualizar dependencias
uv sync
```

## Recursos

- **Dataset:** [Meta Data for Good - Forest Datasets](https://dataforgood.facebook.com/)
- **Documentación AWS CLI:** https://docs.aws.amazon.com/cli/
- **uv Documentation:** https://docs.astral.sh/uv/
- **QGIS:** https://qgis.org/

## Notas

- El sistema de tiles usa **QuadKeys** de Microsoft para identificar teselas geográficas
- Los archivos descargados pueden ser grandes (varios GB para regiones extensas)
- El bucket es público pero requiere el flag `--no-sign-request` en AWS CLI
- Puedes consultar `chm_contents.txt` para ver todos los tiles disponibles y sus tamaños antes de descargar
- Los tiles con tamaño de exactamente 21,168,737 bytes suelen indicar áreas sin datos forestales
