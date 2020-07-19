
## Bioinformatics

#### Convert Gene Symbol in Ensembl and Entrez IDs

```r
#Genome wide annotation for Human  
library(org.Hs.eg.db)  
ENTREZID <- mapIds(org.Hs.eg.db, df$SYMBOL, 'ENTREZID', 'SYMBOL')
```

Use `ALIAS` in place of `SYMBOL` if you are not shure that you are using RefSeq/Hugo approved symbols

#### Convert Ensembl ID retriving data from BioMart
more alternatives (
https://shiring.github.io/genome/2016/10/23/AnnotationDbi

**Look-up ENSEMBL gene IDs:**

```r
ex <- c("ENSG00000215417", "ENSG00000224078", "ENSG00000198366",
"ENSG00000196176", "ENSG00000166012", "ENSG00000158406",
"ENSG00000196787")
mart <- useMart("ensembl", dataset = "hsapiens_gene_ensembl")
gene2genomeEx <- getBM(values = ex,
  filters = "ensembl_gene_id",
  mart = mart,
  attributes = c("ensembl_gene_id", "entrezgene_id",
    "hgnc_symbol", "external_gene_name",
    "description", "chromosome_name",
    "strand"))
gene2genomeEx

  ensembl_gene_id entrezgene hgnc_symbol external_gene_name
1 ENSG00000158406       8365    HIST1H4H           HIST1H4H
2 ENSG00000166012      79101       TAF1D              TAF1D
3 ENSG00000196787       8969   HIST1H2AG          HIST1H2AG
4 ENSG00000215417     407975     MIR17HG            MIR17HG
5 ENSG00000224078       3653      SNHG14             SNHG14
```

You can use also `"ensembl_gene_id_version"` for the format ENSG00000223972.4

**Look-up 'external' gene names:**

```r
ex <- c("ACTN4","TUBA1B","ACTN1","TP53")
mart <- useMart("ensembl", dataset = "hsapiens_gene_ensembl")
gene2genomeEx <- getBM(values = ex,
  filters = "external_gene_name",
  mart = mart,
  attributes = c("external_gene_name", "entrezgene_id",
    "hgnc_symbol", "description",
    "chromosome_name", "strand"))
gene2genomeEx

  external_gene_name entrezgene hgnc_symbol
1               TP53       7157        TP53
2              ACTN4         81       ACTN4
3             TUBA1B      10376      TUBA1B
4              ACTN1         87       ACTN1
5              ACTN4         81       ACTN4
                                            description chromosome_name strand
1 tumor protein p53 [Source:HGNC Symbol;Acc:HGNC:11998]              17     -1
2     actinin alpha 4 [Source:HGNC Symbol;Acc:HGNC:166]              19      1
3  tubulin alpha 1b [Source:HGNC Symbol;Acc:HGNC:18809]              12     -1
4     actinin alpha 1 [Source:HGNC Symbol;Acc:HGNC:163]              14     -1
5     actinin alpha 4 [Source:HGNC Symbol;Acc:HGNC:166]  CHR_HG26_PATCH      1
```
## Functional Analysis
### Over Representation Analysis
https://yulab-smu.github.io/clusterProfiler-book/
```r
library(clusterProfiler)
ORA <- enrichGO(gene = overexpressed_genes, 
                universe = background_genes,
                keyType = "ENTREZID",
                OrgDb = org.Hs.eg.db, 
                ont = "BP", 
                pAdjustMethod = "BH", 
                qvalueCutoff = 0.05, 
                readable = TRUE)

ORA_summary <- data.frame(ORA)

barplot(ORA)
dotplot(ORA)


ORA <- enrichKEGG(gene = overexpressed_genes, 
                universe = background_genes,
                keyType = "uniprot",
                organism = 'hsa',
                pAdjustMethod = "BH", 
                qvalueCutoff = 0.05, 
                readable = TRUE)
```
`readable = TRUE` associates Hugo Gene Names to ENTREZ IDs to make cnetplot and summary tables readable. This argument is not available for the enrichKEGG function.

### Functional Analysis of multiple samples
```r
# sample is a list of vector containing ENTREZ IDs
sample <- list('CA1'=mut.CA1, 'LM'=mut.LM, 'LUC4'=mut.LUC4, 'SCC4'=mut.SCC4, 'SCC9'=mut.SCC9, 'SCC15'=mut.SCC15, 'SCC25'=mut.SCC25)

ORAKegg <- compareCluster(pat, fun="enrichKEGG",
                     organism="hsa", pvalueCutoff=0.1, pAdjustMethod = "BH")
dotplot(ORAKegg)
cnetplot(ORAKegg)
emapplot(ORAGO, categorySize="pvalue", foldChange=geneList)

```
multiple samples emapplot requires the latest GitHub version of ClusterProfiler:
```r
devtools::install_github('https://github.com/YuLab-SMU/clusterProfiler')
```

## Identify genes in genomic ranges
```r
library(GenomicRanges)
# generate 10 random segments
regions.gr <- GRanges(seqnames=Rle(rep(1,10)), ranges=IRanges(start=sample(1:1000,10), width=200), strand=Rle(rep("*",10)))
regions.df <- as.data.frame(regions.gr)
# generate 5 random genes
fiveGeneNames <- paste(sample(letters,5), sample(1:100,5), sep="")
genes.gr <- GRanges(seqnames=Rle(rep(1,5)), ranges=IRanges(start=sample(1:1000,5), width=100, names=fiveGeneNames), strand=Rle(sample(c("+","-"),5, replace=T)))
genes.df <- as.data.frame(genes.gr)
genes.df$name <- rownames(genes.df)
# what regions overlap what genes?
overlapGenes <- findOverlaps(query = genes.gr, subject = regions.gr, type='within')

# Return any genes with an overlap.
# Convert the resulting "Hits" object to a data frame
# and use it as an index
overlapGenes.df <- as.data.frame(overlapGenes)

# queryHits are the Genes index
# subject hits are the regions
overlapGenes.df$Genes <- genes.df[overlapGenes.df$queryHits, 'name']
```

How to download a bed reference file for the human genome
```sh
#!/bin/bash
wget -qO- ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_34/GRCh37_mapping/gencode.v34lift37.annotation.gff3.gz \
    | gunzip --stdout - \
    | awk '$3 == "gene"' - \
    | convert2bed -i gff - \
    > genes.bed
```
