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
