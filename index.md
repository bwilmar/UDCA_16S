# UDCA_16S

Pipeline reproducible para análisis de datos 16S usando QIIME2.

## Navegación

- [Objetivo del análisis](#objetivo-del-análisis)
- [Workflow](#workflow)
- [Preparación de archivos](#preparación-de-archivos)
- [Importación a QIIME2](#importación-a-qiime2)
- [DADA2](#dada2)
- [Asignación taxonómica](#asignación-taxonómica)
- [Diversidad alfa y beta](#diversidad-alfa-y-beta)
- [Scripts](#scripts)
- [Repositorio](#repositorio)

## Objetivo del análisis

Este repositorio documenta el procesamiento bioinformático de secuencias 16S rRNA.

## Workflow

![Workflow](workflow.png)

## Preparación de archivos

Aquí van los comandos para renombrar los FASTQ.

## Importación a QIIME2

Aquí van los comandos `qiime tools import`.

## DADA2

Aquí van los comandos `qiime dada2 denoise-paired`.

## Asignación taxonómica

Aquí van los comandos de clasificación taxonómica.

## Diversidad alfa y beta

Aquí van los análisis de diversidad.

## Scripts

- [Script principal V3_V4](V3_V4)
- [README en GitHub](https://github.com/bwilmar/UDCA_16S)

## Repositorio

Sitio generado con GitHub Pages.

Copy and rename files to match QIIME2's expected Casava 1.8

```
mkdir qiime_import
cp *.fastq.gz qiime_import
cd qiime_import

i=1
for s in A1 A2 A3 C1 C2 C3 C4 D1 D2 D3 D4; do
  mv ${s}_cleaned_R1.fastq.gz ${s}_S${i}_L001_R1_001.fastq.gz
  mv ${s}_cleaned_R2.fastq.gz ${s}_S${i}_L001_R2_001.fastq.gz
  i=$((i+1))
done

````
#Import to Qiime2

````
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path /home/wbotellos/UDCA/data/raw/20210331_Cruz_Elena_raw_data_proyecto/qiime_import \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path demux_PE.qza

````
Resumen y calidad 
````
qiime demux summarize \
  --i-data demux_PE.qza \
  --o-visualization demux_PE.qzv
````
Remover amplicon primers 
````
qiime cutadapt trim-paired \
  --i-demultiplexed-sequences demux_PE.qza \
  --p-cores 16 \
  --p-front-f CCTAYGGGRBGCASCAG \
  --p-front-r GACTACHVGGGTATCTAATCC \
  --p-discard-untrimmed \
  --o-trimmed-sequences trimmed_PE.qza \
  --verbose \
  &> primer_trimming.log
````
Verificar 

````
qiime demux summarize \
  --i-data trimmed_PE.qza \
  --o-visualization trimmed_PE.qzv
````
#Realice submuiestreo para verificar si es posible realizar un merge

````
qiime demux subsample-paired \
  --i-seqs trimmed_PE.qza \
  --p-fraction 0.07 \
  --o-subsampled-seqs trimmed_PE_sub.qza

qiime demux summarize \
  --i-data trimmed_PE_sub.qza \
  --o-visualization trimmed_PE_sub.qzv
````
Obtuve un merge >80% lo cual puede ser consistente, vamos a realizar con todas las muestras:

````
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs trimmed_PE.qza \
  --p-trunc-len-f 260 \
  --p-trunc-len-r 220 \
  --p-trim-left-f 0 \
  --p-trim-left-r 0 \
  --p-n-threads 16 \
  --output-dir dada2_paired_full \
  --verbose \
  &> dada2_paired_full.log

````
Verificamos la colmna merged 

````

qiime metadata tabulate \
  --m-input-file dada2_paired_full/denoising_stats.qza \
  --o-visualization dada2_paired_full/denoising_stats.qzv

qiime tools view dada2_paired_full/denoising_stats.qzv

````


