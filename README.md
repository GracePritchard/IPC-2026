# IPC-2026
proyecto ejemploIPC 2026
 ## Estos es un subtiutlo 
 Cambio en rama secundaria 
 

> [!WARNING]
> **Proyecto docente: simulación con fines exclusivamente académicos.**
>
> Este repositorio se ha creado para la asignatura _Introducción a la Programación Científica_ del Máster en Bioinformática de la UNIR, con el objetivo de practicar el uso de Git y GitHub. **No es un pipeline funcional ni validado.**
>
> Los scripts son esqueletos (pueden estar vacíos) y los resultados son ficheros de ejemplo. **No debe usarse para análisis científicos ni clínicos.**


[![Genoma de referencia](https://img.shields.io/badge/genoma-hg38-blue)](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000001405.26/)
[![Datos](https://img.shields.io/badge/datos-WGS-green)](data/)
![Variantes](https://img.shields.io/badge/variantes-SNV%20%7C%20INDEL-purple)
[![Entorno](http://img.shields.io/badge/run%20with-conda-3EB049?labelColor=000000&logo=anaconda)](https://docs.conda.io/en/latest/)
[![Scripts](https://img.shields.io/badge/scripts-Bash-4EAA25?logo=gnubash&logoColor=white)](scripts/)
![Estado](https://img.shields.io/badge/estado-proyecto%20docente-orange)
[![Licencia: MIT](https://img.shields.io/badge/licencia-GPLv3-yellow.svg)](LICENSE)

```mermaid
flowchart LR
    FASTQ[("FASTQ<br/>lecturas en bruto")]
    REF[("hg38.fa<br/>referencia")]
    DBSNP[("dbSNP")]
    CACHE[("Caché de VEP")]

    S1["01 · FastQC<br/>control de calidad"]
    S2["02 · BWA-MEM2<br/>alineamiento"]
    S3["03 · Picard<br/>métricas del BAM"]
    S4["04 · HaplotypeCaller<br/>llamada de variantes"]
    S5["05 · VariantFiltration<br/>filtrado"]
    S6["06 · VEP<br/>anotación"]

    BAM[("BAM")]
    RAW[("raw.vcf.gz")]
    FIL[("filtered.vcf.gz")]
    ANN[("annotated.vcf.gz")]

    FASTQ --> S1
    FASTQ --> S2
    REF --> S2
    S2 --> BAM
    BAM --> S3
    BAM --> S4
    REF --> S4
    DBSNP --> S4
    S4 --> RAW
    RAW --> S5
    S5 --> FIL
    FIL --> S6
    CACHE --> S6
    S6 --> ANN

    classDef paso fill:#dbeafe,stroke:#1d4ed8,color:#000
    classDef dato fill:#fef9c3,stroke:#a16207,color:#000
    classDef final fill:#dcfce7,stroke:#15803d,color:#000

    class S1,S2,S3,S4,S5,S6 paso
    class FASTQ,REF,DBSNP,CACHE,BAM,RAW,FIL dato
    class ANN final
```


## Instalación

### Requisitos

- [Conda](https://docs.conda.io/) (o [Miniconda](https://docs.conda.io/en/latest/miniconda.html))
- Se recomienda usar Linux (algunas herramientas, como BWA-MEM2, están pensadas para Linux x86_64 )
- Espacio en disco para la referencia, la caché de VEP y los resultados (varios GB por muestra)

### Pasos

1. Clona el repositorio:

   ```bash
   git clone https://github.com/<usuario>/wgs-germline-variant-calling-IPC-2026.git
   cd wgs-germline-variant-calling-IPC-2026
   ```

2. Crea el entorno con todas las herramientas:

   ```bash
   conda env create -f envs/environment.yml
   ```

3. Actívalo:

   ```bash
   conda activate wgs-germline-variant-calling
   ```

4. Comprueba que las herramientas están disponibles:

   ```bash
   fastqc --version
   bwa-mem2 version
   samtools --version
   gatk --version
   vep --help
   ```

## Preparación de los datos

Los datos no se incluyen en el repositorio (ver [`data/`](data/)). Antes de ejecutar el pipeline hay que:

1. Colocar los FASTQ de cada muestra en `data/raw/`.
2. Descargar el genoma hg38 en `data/reference/` y generar sus índices:

   ```bash
   samtools faidx data/reference/hg38.fa
   gatk CreateSequenceDictionary -R data/reference/hg38.fa
   bwa-mem2 index data/reference/hg38.fa
   ```

3. Descargar dbSNP y la caché de VEP en `data/reference/`:

   ```bash
   vep_install -a cf -s homo_sapiens -y GRCh38 -c data/reference/vep_cache
   ```

4. Definir las muestras en `config/samples.tsv` y las rutas en `config/config.yaml`
   (ver [Configuración](#configuración)).

## Uso

> [!NOTE]
> En esta versión los scripts son esqueletos, así que los comandos siguientes muestran **cómo se ejecutaría el pipeline**, no producen resultados reales.

Los scripts se ejecutan **desde la raíz del repositorio**, en orden, y reciben el identificador de la muestra (`sample_id`) como argumento:

```bash
SAMPLE=sample1

bash scripts/01_fastq_qc.sh          $SAMPLE   # control de calidad de las lecturas
bash scripts/02_alignment.sh         $SAMPLE   # alineamiento contra hg38
bash scripts/03_bam_qc.sh            $SAMPLE   # métricas del alineamiento
bash scripts/04_variant_calling.sh   $SAMPLE   # llamada de variantes
bash scripts/05_variant_filtering.sh $SAMPLE   # filtrado de variantes
bash scripts/06_annotation.sh        $SAMPLE   # anotación con VEP
```

Para procesar todas las muestras de `config/samples.tsv`:

```bash
tail -n +2 config/samples.tsv | cut -f1 | while read -r SAMPLE; do
    for script in scripts/0*.sh; do
        bash "$script" "$SAMPLE"
    done
done
```

### Salidas esperadas

Tras ejecutar el pipeline con `sample1`, se generarían:

| Paso | Fichero | Carpeta |
|---|---|---|
| 01 | `sample1_R1_fastqc.html`, `sample1_R2_fastqc.html` | `results/qc/` |
| 02 | `sample1.bam`, `sample1.bam.bai` | `results/bam/` |
| 03 | `sample1_metrics.txt` | `results/qc/` |
| 04 | `sample1.raw.vcf.gz` | `results/vcf/` |
| 05 | `sample1.filtered.vcf.gz` | `results/vcf/` |
| 06 | `sample1.annotated.vcf.gz` | `results/vcf/` |

El resultado final es `results/vcf/sample1.annotated.vcf.gz`.
