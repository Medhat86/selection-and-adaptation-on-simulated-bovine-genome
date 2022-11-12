
# <img src="https://github.com/Medhat86/Ghat/blob/master/GOE_Logo_Quer_IPC_Farbe_RGB.png" />


# Selection and Adaptation on Simulated Bovine Genome

This repository houses code and data (simulated bovine genome) to identify selection on quantitative traits. In this repository, we have divided files into sub directories as outlined below. For analyses performed using GWDG Scientific Compute Cluster, we have deposited both bash files and outputs.

We generated a simulated cattle data set with divergent selection to further test and demonstrate the efficacy of Ghat. This allowed us to evaluate how the Ghat package per-forms in terms of different genetic architectures and experimental parameters, which are never perfectly known using real data. In this analysis, we assessed the impact of trait heritability (h2), sample size (n), trait polygenicity (number of QTL (nQTL)), and marker density (MD) on the performance of Ghat. 


## Examples


### Example-1 Both SNP effects and change in allele frequency are known


```r

rm(list=ls())

library(BGLR)
source("Ghat2.R")

Pvals.res <- c()
Ghat.res <- c()
Cor.res <- c()

for(sim in 1:100){
  iter <- sprintf("%03i",sim)
  directory="/home/uni08/mahmoud1/Ghat_sim/r_h1"
  
  #load genotypes and phenotypes
  
  pheno   <- read.table(paste(directory, "/Line_1_data_",iter,".txt",sep=""),
                        header=T,stringsAsFactors=F)
  map     <- read.table(paste(directory, "/lm_mrk_",iter,".txt",sep=""),
                        header=T,stringsAsFactors=F)
  geno <- read.table(paste(directory, "/Line_1_mrk_",iter,".txt",sep=""),
                     header=F,stringsAsFactors=F,skip=1,sep="", 
                     colClasses=c("numeric","character")) # read genos as characters

  
  #load allele frequencies
  freqs<-read.table(paste(directory, "/Line_1_freq_mrk_",iter,".txt",sep=""),
                    header=T,fill=T,stringsAsFactors=F)
  freqs_15    <-  freqs[freqs$Gen == 15,] 
  freqs_20    <-  freqs[freqs$Gen == 20,]
  
  ### Manipulate genotypes by coding to -1,0,1 and so that markers are columns,
  #individuals are rows.
  
  gen <- matrix(NA,nrow=nrow(map),ncol=nrow(geno))
  gen <- as.data.frame(gen)
  names(gen) <- geno[,1]
  for(i in 1:nrow(geno)){
    #print(i)
    tmp <- as.numeric(unlist(strsplit(geno[i,2],split="")))
    tmp[which(tmp == 0)] <- -1
    tmp[which(tmp == 3 | tmp ==4)] <- 0
    tmp[which(tmp==2)] <- 1
    gen[,i] <- tmp
  }
  gen<-t(gen)
  gc()
  
  
  ##### Estimate allel efeects 
  
  g <- cbind(row.names(gen),data.frame(gen +1))
  g$`row.names(gen)`<- NULL
  
  nIter=600; burnIn=100
  
  fmBC=BGLR(pheno$Phen,ETA=list( list(X=g,model='BayesC')), 
            nIter=nIter,burnIn=burnIn,saveAt='bc_')
  
  se <- data.frame(fmBC$ETA[[1]]$b)
  
  ### Calculate allele frequencies
  # Generation 15
  names(freqs_15)[4] <- "Allele1"
  names(freqs_15)[5]<- "Allele2"
  freqs_15$Allele2[which(substr(freqs_15$Allele1,1,1)==2)] <- "2:1.00000" 
  # put this in the spot for the second allele
  freqs_15$Allele1[which(substr(freqs_15$Allele1,1,1)==2)] <- "1:0.00000" 
  freqs_15$Allele2[which(substr(freqs_15$Allele1,3,3)==1)] <- "2:0.00000" ##
  freqs_15$Allele1 <- as.numeric(substr(freqs_15$Allele1,3,nrow(map)))
  freqs_15$Allele2 <- as.numeric(substr(freqs_15$Allele2,3,nrow(map)))
  # Generation 20
  names(freqs_20)[4]<- "Allele1"
  names(freqs_20)[5]<- "Allele2"
  freqs_20$Allele2[which(substr(freqs_20$Allele1,1,1)==2)] <- "2:1.00000" 
  # put this in the spot for the second allele
  freqs_20$Allele1[which(substr(freqs_20$Allele1,1,1)==2)] <- "1:0.00000" 
  freqs_20$Allele2[which(substr(freqs_20$Allele1,3,3)==1)] <- "2:0.00000"  ##
  freqs_20$Allele1 <- as.numeric(substr(freqs_20$Allele1,3,nrow(map)))
  freqs_20$Allele2 <- as.numeric(substr(freqs_20$Allele2,3,nrow(map)))
  #Calculate change
  change2<-freqs_20$Allele2-freqs_15$Allele2
  
  ##### load SNP effects
  
  
  test<-  Ghat2(effects=se$fmBC.ETA..1...b, change=change2, 
               method = "scale", perms = 1000,
               num_eff = 2000)
  
  Ghat.res[as.numeric(iter)]  <- test$Ghat
  Pvals.res[as.numeric(iter)] <- test$p.val
  Cor.res[as.numeric(iter)]   <- test$Cor
}

### Report the results

h1_s <- cbind(Ghat.res, Pvals.res, Cor.res)
save(h1_s, file = "h1_s.RData")

## ############################
## ############################
## ############################

### Estimating Ghat for the Controlled Populations.


setwd("~/Ghat_sim")
rm(list=ls())

library(BGLR)
source("Ghat2.R")

Pvals.res <- c()
Ghat.res <- c()
Cor.res <- c()

for(sim in 1:100){
  iter <- sprintf("%03i",sim)
  directory="/home/uni08/mahmoud1/Ghat_sim/r_h1"
  
  #load genotypes and phenotypes
  
  pheno   <- read.table(paste(directory, "/Line_2_data_",iter,".txt",sep=""),
                        header=T,stringsAsFactors=F)
  map     <- read.table(paste(directory, "/lm_mrk_",iter,".txt",sep=""),
                        header=T,stringsAsFactors=F)
  geno <- read.table(paste(directory, "/Line_2_mrk_",iter,".txt",sep=""),
                     header=F,stringsAsFactors=F,skip=1,sep="", 
                     colClasses=c("numeric","character")) # read genos as characters
  
  
  #load allele frequencies
  freqs<-read.table(paste(directory, "/Line_2_freq_mrk_",iter,".txt",sep=""),
                    header=T,fill=T,stringsAsFactors=F)
  freqs_15    <-  freqs[freqs$Gen == 15,] 
  freqs_20    <-  freqs[freqs$Gen == 20,]
  
  ### Manipulate genotypes by coding to -1,0,1 and so that markers are columns,
  #individuals are rows.
  
  gen <- matrix(NA,nrow=nrow(map),ncol=nrow(geno))
  gen <- as.data.frame(gen)
  names(gen) <- geno[,1]
  for(i in 1:nrow(geno)){
    #print(i)
    tmp <- as.numeric(unlist(strsplit(geno[i,2],split="")))
    tmp[which(tmp == 0)] <- -1
    tmp[which(tmp == 3 | tmp ==4)] <- 0
    tmp[which(tmp==2)] <- 1
    gen[,i] <- tmp
  }
  gen<-t(gen)
  gc()
  
  
  ##### Estimate allel efeects 
  
  g <- cbind(row.names(gen),data.frame(gen +1))
  g$`row.names(gen)`<- NULL
  
  nIter=600; burnIn=100
  
  fmBC=BGLR(pheno$Phen,ETA=list( list(X=g,model='BayesC')), 
            nIter=nIter,burnIn=burnIn,saveAt='bc_')
  
  se <- data.frame(fmBC$ETA[[1]]$b)
  
  ### Calculate allele frequencies
  # Generation 15
  names(freqs_15)[4] <- "Allele1"
  names(freqs_15)[5]<- "Allele2"
  freqs_15$Allele2[which(substr(freqs_15$Allele1,1,1)==2)] <- "2:1.00000" 
  # put this in the spot for the second allele
  freqs_15$Allele1[which(substr(freqs_15$Allele1,1,1)==2)] <- "1:0.00000" 
  freqs_15$Allele2[which(substr(freqs_15$Allele1,3,3)==1)] <- "2:0.00000" ##
  freqs_15$Allele1 <- as.numeric(substr(freqs_15$Allele1,3,nrow(map)))
  freqs_15$Allele2 <- as.numeric(substr(freqs_15$Allele2,3,nrow(map)))
  # Generation 20
  names(freqs_20)[4]<- "Allele1"
  names(freqs_20)[5]<- "Allele2"
  freqs_20$Allele2[which(substr(freqs_20$Allele1,1,1)==2)] <- "2:1.00000" 
  # put this in the spot for the second allele
  freqs_20$Allele1[which(substr(freqs_20$Allele1,1,1)==2)] <- "1:0.00000" 
  freqs_20$Allele2[which(substr(freqs_20$Allele1,3,3)==1)] <- "2:0.00000"  ##
  freqs_20$Allele1 <- as.numeric(substr(freqs_20$Allele1,3,nrow(map)))
  freqs_20$Allele2 <- as.numeric(substr(freqs_20$Allele2,3,nrow(map)))
  #Calculate change
  change2<-freqs_20$Allele2-freqs_15$Allele2
  
  ##### load SNP effects
  
  
  test<-  Ghat2(effects=se$fmBC.ETA..1...b, change=change2, 
                method = "scale", perms=1000,
                num_eff = 2000)
  
  Ghat.res[as.numeric(iter)]  <- test$Ghat
  Pvals.res[as.numeric(iter)] <- test$p.val
  Cor.res[as.numeric(iter)]   <- test$Cor
}

### Report the results

h1_c <- cbind(Ghat.res, Pvals.res, Cor.res)
save(h1_c, file = "h1_c.RData")




```


Please visit [https://github.com/Medhat86/Ghat](https://cran.r-project.org/web/packages/Ghat/Ghat.pdf) for documentation and examples.

## Citation

- In case you want / have to cite my package, please use `citation('Ghat')` (https://cran.r-project.org/web/packages/Ghat/index.html) for citation information. 

- Since core functionality of package depends on the [Ghat-Method](https://cran.r-project.org/package=ggplot2), consider citing this refernce as well: "Tim Beissinger, Jochen Kruppa, David Cavero, Ngoc-Thuy Ha, Malena Erbe, Henner Simianer, A Simple Test Identifies Selection on Complex Traits, Genetics, Volume 209, Issue 1, 1 May 2018, Pages 321–333, https://doi.org/10.1534/genetics.118.300857"

