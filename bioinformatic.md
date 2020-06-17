
## Bioinformatics

#### Convert Gene Symbol in Ensembl and Entrez IDs

```r
#Genome wide annotation for Human  
library(org.Hs.eg.db)  
ENTREZID <- mapIds(org.Hs.eg.db, df$SYMBOL, 'ENTREZID', 'SYMBOL')
```

Use ALIAS in place of SYMBOL if you are not shure that you are using RefSeq/Hugo approved symbols

#### Convert Ensembl ID retriving data from BioMart

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

You can use also "ensembl_gene_id_version" for the format ENSG00000223972.4

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
#### Gene Ontology
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
readable = TRUE associates Gene Symbol Names to ENTREZ ID; this is not available with enrichKEGG

