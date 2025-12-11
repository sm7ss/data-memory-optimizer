# 🚀 Data Memory Optimizer

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![Polars](https://img.shields.io/badge/Polars-Fast__DataFrames-red.svg)](https://pola.rs/)
[![PyArrow](https://img.shields.io/badge/PyArrow-Efficient__Memory-orange.svg)](https://arrow.apache.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Una herramienta inteligente de optimización de memoria que analiza automáticamente archivos de datos (CSV/Parquet) y determina la estrategia óptima de procesamiento (`eager`, `lazy`, o `streaming`) basada en los recursos del sistema.

## 🔍 El Problema que Resuelvo

Como Data Engineer, constantemente enfrento la pregunta: **¿Cómo procesar este archivo sin saturar la memoria del sistema?** 

Esta herramienta responde automáticamente esa pregunta analizando:
- 📊 **Overhead específico** de CSV vs Parquet
- 💾 **Recursos disponibles** del sistema
- 📏 **Tamaño y estructura** de los datos
- ⚡ **Estrategia óptima**: eager, lazy, o streaming

## 🎯 Características Principales

### 📈 **Análisis Inteligente de Archivos**

- Estimación precisa de overhead por tipo de columna
- Análisis de strings y su impacto en memoria
- Soporte para CSV y Parquet con algoritmos especializados

### 💡 **Decision Making Automático**

- Basado en ratio memoria estimada / memoria disponible
- Umbrales optimizados por experiencia empírica
- Considera margen de seguridad para el sistema operativo

### 📊 **Métricas y Resultados Medibles**

- Ratio de utilización de memoria
- Overhead estimado por tipo de dato
- Recomendación cuantificada y justificada

## 🚀 Instalación y Uso Rápido

### **Instalación**

```bash
git clone https://github.com/sm7ss/data-memory-optimizer.git
cd data-memory-optimizer
pip install -r requirements.txt
```

### **Uso Básico**

```python 
from src.path_desicion_maker import PipelineEstimatedSizeFiles

# Analizar un archivo
analyzer = PipelineEstimatedSizeFiles("datos.csv")
result = analyzer.estimated_size_file()

print(f"Estrategia recomendada: {result['decision']}")
print(f"Ratio memoria: {result['ratio']}")
print(f"Memoria estimada: {result['memoria_total_estimada_gb']} GB")
```

## 📊 Resultados de Pruebas (Métricas Reales)

### 📈 **Archivo Parquet Grande**

```python 
{
    'ratio': 1.325,
    'overhead_estimado': 1.36,
    'safety_memory': 4.667,
    'archivo_descomprimido_gb': 5.612,
    'total_de_filas_gb': 244673551,
    'memoria_total_estimada_gb': 7.632,
    'memoria_disponible': 10.426,
    'total_memory': 15.555,
    'decision': 'lazy'  # ✅ Recomendación óptima
}
```

### 📊 **Archivo CSV Pequeño**

```python 
{
    'ratio': 0.0,
    'total_rows': 89,
    'bytes_por_columna': 148.0,
    'safety_memory': 4.667,
    'memoria_total_estimada_gb': 0.0,
    'memoria_disponible': 10.424,
    'total_memory': 15.555,
    'decision': 'eager'  # ✅ Puede cargarse completo
}
```

## 🔧 Algoritmos Implementados

### **CSV Overhead Analysis**

```python 
class CsvOverhead:
    def string_csv_overhead(self) -> float:
        # Análisis de overhead específico de strings en CSV
        avg_len = sum(self.frame_sample[col].str.len_bytes().median() 
                     for col in self.str_columns) / len(self.str_columns)
        
        # Asignación de factor basado en longitud promedio
        if avg_len <= 1: return 2.0
        elif avg_len <= 5: return 1.8
        elif avg_len <= 10: return 1.6
        # ... lógica optimizada
```

### **Parquet Overhead Analysis**

```python 
class ParquetOverheadEstimator:
    def string_overhead(self) -> float:
        # Cálculo especializado para strings en Parquet
        avg_string_len = sum([
            sample_median[col].str.len_bytes().sum() 
            for col in string_columns
        ]) / len(string_columns)
        
        base = 1.0 + 4.0 / avg_string_len
        # Ajustes basados en benchmarks reales
```

### **Decision Making Engine**

```python 
class FileSizeEstimator:
    def estimate_csv_size(self, csv_overhead_class) -> Dict[str, Any]:
        # Cálculo de ratio crítico
        ratio = estimated_memory / usable_ram
        
        # Lógica de decisión optimizada
        if ratio <= 0.65: return 'eager'
        elif ratio <= 2.0: return 'lazy'
        else: return 'streaming'
```

## 🎯 Lógica de Decisión

### 📈 **Thresholds Optimizados**

| Ratio (Estimado/Disponible) | Estrategia | Justificación                          |
|-----------------------------|------------|----------------------------------------|
| **≤ 0.65**                  | eager      | Memoria suficiente para carga completa |
| **0.65 - 2.0**              | lazy	   | Procesamiento por lotes recomendado    |
| **> 2.0**	                  | streaming  | Requiere procesamiento incremental     |

### 🛡️ **Safety Margins**

- 30% de memoria reservada para sistema operativo
- Análisis por tipo de dato con overhead específico
- Consideración de strings como mayor impacto

### 📦 **Dependencias**

polars>=0.19.0
pyarrow>=12.0.0
psutil>=5.9.0

## 🏆 Casos de Uso en Producción

### **1. Pipeline de ETL Automatizado**

```python 
# Integración en pipeline de data engineering
def process_file_optimally(file_path: str):
    analyzer = PipelineEstimatedSizeFiles(file_path)
    result = analyzer.estimated_size_file()
    
    if result['decision'] == 'eager':
        return pl.read_csv(file_path)  # Carga completa
    elif result['decision'] == 'lazy':
        return pl.scan_csv(file_path)  # Procesamiento lazy
    else:
        return pl.scan_csv(file_path).collect(streaming=True)  # Streaming
```

### **2. Sistema de Monitoreo de Recursos**

```python 
# Monitoreo proactivo de uso de memoria
class ResourceMonitor:
    def check_file_safety(self, file_path: str):
        result = PipelineEstimatedSizeFiles(file_path).estimated_size_file()
        if result['ratio'] > 1.5:
            logger.warning(f"Archivo {file_path} puede saturar memoria")
            return False
        return True
```

### **3. Optimización de Queries**

```python 
# Selección automática de estrategia de query
def optimize_query(file_path: str, query):
    result = PipelineEstimatedSizeFiles(file_path).estimated_size_file()
    
    if result['decision'] == 'streaming':
        return execute_streaming_query(file_path, query)
    else:
        return execute_standard_query(file_path, query)
```

## 🔬 Detalles Técnicos Avanzados

### **Overhead por Tipo de Dato**

| Tipo de Dato	    | CSV Overhead | Parquet Overhead | Justificación        |
|-------------------|--------------|------------------|----------------------|
| **Int8/16/32/64** | 1.4-1.55     | 1.2-1.35         |	Overhead de encoding |
| **Float32/64**	| 1.6-1.65	   | 1.4-1.45         |	Precisión decimal    |
| **String**	    | 1.1-2.0	   | 1.05-2.2         |	Longitud variable    |
| **Boolean**	    | 2.5	       | 2.0	          | Ineficiente en texto |
| **DateTime**	    | 1.85	       | 1.55	          | Formato timestamp    |

### **Fórmulas de Estimación**

```bash
# Memoria Estimada CSV
memoria_csv = (filas × overhead_promedio × bytes_por_columna) / 1024³

# Memoria Estimada Parquet  
memoria_parquet = (overhead_promedio × tamaño_descomprimido) / 1024³

# Ratio de Decisión
ratio = memoria_estimada / (memoria_disponible - margen_seguridad)
```

## 🤝 Contribución

¡Contribuciones son bienvenidas! Si tienes ideas para mejorar el algoritmo o agregar nuevas características:

1. Haz fork del proyecto
2. Crea una rama para tu feature (git checkout -b feature/mejora-algoritmo)
3. Commit tus cambios (git commit -m 'Agregar soporte para formato X')
4. Push a la rama (git push origin feature/mejora-algoritmo)
5. Abre un Pull Request

## 👩‍💻 Sobre Este Proyecto

Este proyecto nació de la necesidad práctica de o**ptimizar el uso de memoria en pipelines de data engineering**. Como Data Engineer, constantemente enfrentaba el dilema de elegir entre eager, lazy y streaming sin métricas concretas.

Las **métricas presentadas son reales** y representan pruebas con archivos de diferentes tamaños y complejidades. Cada decisión está respaldada por análisis cuantitativo y validación empírica.

**¿Preguntas o sugerencias?** ¡No dudes en abrir un issue!
