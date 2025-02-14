---
title: "cw 7"
output: html_document
date: "2024-11-25"
---

```{r}
BiocManager::install(c("GenomicFeatures", "AnnotationDbi", 
"ShortRead", "Biostrings","ggplot2", "Rsubread", "Rqc","GenomicAlignments"))
```

```{r}
library(GenomicFeatures) 
library(AnnotationDbi)
library(ShortRead)
library(Biostrings)
library(ggplot2)
library(Rsubread)
library (Rqc)
library(GenomicAlignments)
```

### Mapowanie do genomu referencyjnego

**Genom referencyjny** - standardowa sekwencja DNA ktora sluzy jako punkt odniesienia do wszystkich analiz

-   U ssaków zazwyczaj osobnik o wysokiej homozygotyczności

-   pozwala na identyfikacje wariantów genetycznych tj. SNP, insercje i delecje

**Cele mapowania sekwencji :**

-   lokalizacja odczytów - określenie położenia sekwencji w genomie referencyjnym

**Metody i algorytmy :**

-   algorytmy dopasowania sekwencji --\> dopasowanie wszystkich nt

-   algorytmy dopasowania niedokładnego: pozwalają na pewna liczbę niedopasowań

-   Metody indeksowania genomu: umożliwiają szybkie wyszukiwania sekwencji

**Pokrycie genomu - coverage** - oznacza ile razy dany fragment genomu zosta odczytany podczas sekwencjonowania

**POKRYCIE:**

średnie pokrycie = suma długości wszystkich odczytów podzielona przez długość genomu referencyjnego

-   wysokie pokrycie – powyżej 30

-   30 – dobre pokrycie

-   niskie pokrycie – poniżej 10

### Zimportowanie pliku FASTQ i generowanie raportu QC

```{r}

# Zaimportuj plik FASTQ
fq1 <- readFastq("C:/Users/s207321/Desktop/ćw7/SRR31368982.fastq")

# Przeprowadź analizę jakości
qa_result <- qa(fq1,lane = "1")
report(qa_result, dest="C:/Users/s207321/Desktop/ćw7")
```

**Import genomu referencyjnego**

```{r}
ref_genome <- readDNAStringSet("C:/Users/s207321/Desktop/ćw7/ecoli_genome.fna.gz")
```

**Indeksowanie genomu referencyjnego**

```{r}
#Budowanie indeksu genomu

buildindex(basename = "ecoli_index", reference = "C:/Users/s207321/Desktop/ćw7/ecoli_genome.fna.gz")
```

**Wszystko zostało zapisane w katalogu:**

```         
Index C:\Users\s207321\Documents\ecoli_index was successfully built. 
```

### Mapowanie odczytów do genomu referencyjnego

```{r}
#mapowanie odczytów 
align(index = "ecoli_index",
      readfile1 = "C:/Users/s207321/Desktop/ćw7/SRR31368982.fastq",
      input_format = "FASTQ",
      output_file = "C:/Users/s207321/Desktop/ćw7/aligned_sample.BAM")
```

**Wynik**

![(C:/Users/s207321/Desktop/ćw7/obraz1)](C:/Users/s207321/Desktop/ćw7/obraz1.png)

**Procent odczytów zmapowanych**

```{r}
(713927/1219986)*100
```

**Wynik**: 58.51928

**Procent odczytów niezmapowanych**

```{r}
(506059/1219986)*100
```

**Wynik**: 41.48072

### Możliwe przyczyny niezmapowania odczytów:

-   insercje i delecje (określone jako Indels) lub SNP

### Analiza wyników mapowania

```{r}
#Import pliku BAM - zmmapowanych odczytów do R

aln <- readGAlignments("C:/Users/s207321/Desktop/ćw7/aligned_sample.BAM")
```

### Oblicznie pokrycia genomu

```{r}
#Oblicz pokrycie genomu i zidentyfikuj regiony o najwyższym i najniższym pokryciu.
coverage_data <- coverage(aln)
```

### Wizualizacja pokrycia przy pomocy pakietu ggplot2

```{r}

# Konwersja pokrycia do data frame
cov_df <- as.data.frame(coverage_data[[1]])
cov_df$position <- as.numeric(rownames(cov_df))

# Wykres pokrycia
pdf("C:/Users/s207321/Desktop/ćw7/wykres1.pdf", width = 8, height = 6)

ggplot(cov_df[1:25000, ], aes(x = position, y = value)) +
  geom_line(color = "blue") +
  labs(title = "Pokrycie genomu E. coli",
       x = "Pozycja w genomie",
       y = "Liczba zmapowanych odczytów")
       
dev.off()
```

### Powyższa funkcja nie działa dla windows - stosujemy funkcję alternatywną

```{r}
cov_df <- as.data.frame(coverage_data[[1]])
cov_df$position <- as.numeric(rownames(cov_df))

ggplot(cov_df[1:25000, ], aes(x = position, y = value)) +
  geom_line(color = "blue") +
  labs(title = "Pokrycie genomu E. coli",
       x = "Pozycja w genomie",
       y = "Liczba zmapowanych odczytów")

```

![(C:/Users/s207321/Desktop/ćw7/wykres2)](C:/Users/s207321/Desktop/ćw7/wykres2.png)

### Pokrycie średnie genomu 

```{r}
#Obliczanie pokrycia genomu 
#średnie pokrycie = suma długości wszystkich odczytów/przez długość genomu referencyjnego

mean(cov_df$value) 

#średnie pokrycie genomu = ile razy dany fragment został odczytany podczas sekwencjonowania
```

**Wynik**: 32.02298

### Obliczanie długości sekwencji z pliku FASTQ w nukleotydach :

```{r}
#wczytanie pliku genomu Fastq do R/ import 
# Ścieżka do pliku FASTQ
fastq_file <- "C:/Users/s207321/Desktop/ćw7/SRR31368982.fastq"

# Wczytanie danych z pliku FASTQ
fastq_data <- readFastq(fastq_file)

# Pobranie długości poszczególnych sekwencji
sequence_lengths <- width(sread(fastq_data))

# Oblicz całkowitą długość sekwencji
total_length <- sum(sequence_lengths)

# Wyświetlenie wyniku
cat("Całkowita długość sekwencji w pliku FASTQ:", total_length, "nukleotydów\n")
```
