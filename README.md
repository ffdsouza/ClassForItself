# ClassForItself

This repository contains the script and data for the research project "A Class for Itself? On the Worldviews of the New Tech Elite" by Hilke Brockmann (Jacobs University Bremen), Wiebke Drews (Universität der Bundeswehr München) and John Torpey (CUNY). The paper is currently under review. Do not cite or distribute without authors' permission.

In case of any questions, feel free to contact: wiebke.drews@unibw.de

---
title: "A Class For Itself"
authors: Drews, Wiebke & Brockmann, Hilke
---

###Loading Packages and Twitter Data###

```{r}
#setwd("")

#install.packages("quanteda")
#install.packages("readtext")
#install.packages("tidyverse")
#install.packages("httr")
#install.packages("ggplot2")
#install.packages("devtools")
#devtools::install_github("dkahle/ggmap", ref = "tidyup")
#install.packages("mapproj")
#install.packages("maps")

library(quanteda)
library(readtext)
library(dplyr)
library(tidyverse)
library(httr)
library(ggplot2)
library(ggmap)
library(mapproj)
library(maps)

#Reading in Twitter Techies
Twitter_Tech <- readtext(file="TwitterTechSES.csv", text_field="text.1")

#Reading in Twitter Population
Twitter_Pop <- readtext(file="US_Final.csv", text_field="text.1")
```

###World Map of Twitter Techies###

```{r}
#Set your API Key (replace with your own)
ggmap::register_google(key = "XYZ")

#Make residence dataframe
residence <- distinct(Twitter_Tech, Residence)
residence_df <- as.data.frame(residence)
#names(residence_df)

#Exclude the missing/NA observations
residence_df_drop <- residence_df %>%
na.omit()		
dim(residence_df_drop)
#Exclude row 8 (no location)
residence_df_drop[-c(8), ]
residence_df_drop <- residence_df_drop[-c(8),,drop=F]

#Rename misspelled places
residence_df_drop$Residence[residence_df_drop$Residence == "Woodside, CA"] <- "Woodside, California"
residence_df_drop$Residence[residence_df_drop$Residence == "Los Angelos"] <- "Los Angeles"
residence_df_drop$Residence[residence_df_drop$Residence == "Honolulu"] <- "Honolulu, Hawaii"

#mutate geocode to receive long and lat from Google
residence_df_drop <- mutate_geocode(residence_df_drop, Residence)
#names(residence_df_drop)

#rename columns, write CSV of the Geolocations and merge with initial dataset
#rename columns
names(residence_df_drop)[names(residence_df_drop) == 'lon'] <- 'lon_residence'
names(residence_df_drop)[names(residence_df_drop) == 'lat'] <- 'lat_residence'
names(residence_df_drop)

#save geolocation csv
#write.csv("Residence_GeoCode.csv")

#Rename misspelled places
Twitter_Tech$Residence[Twitter_Tech$Residence == "Woodside, CA"] <- "Woodside, California"
Twitter_Tech$Residence[Twitter_Tech$Residence == "Los Angelos"] <- "Los Angeles"
Twitter_Tech$Residence[Twitter_Tech$Residence == "Honolulu"] <- "Honolulu, Hawaii"

#full join the datasets
Twitter_Tech <- full_join(Twitter_Tech, residence_df_drop, by="Residence")
names(Twitter_Tech)

#save geolocation csv
#write.csv(Twitter_Tech, file="Twitter_Tech.csv")

residence <- as_tibble(residence_df_drop)

#Using GGPLOT, plot the Base World Map
mp <- NULL
mapWorld <- borders("world", colour = "grey", fill="beige")
mp <- ggplot() + 
  mapWorld + 
  theme(panel.background = element_rect(fill = "lightblue")) +
  xlab("Longitude") + ylab("Latitude")
mp <- mp+ geom_point(data = residence, aes(x = lon_residence, y = lat_residence),color="black", size=3) 
mp

#ggsave("WorldMap.jpeg", height = 12, width = 21, units = 'cm', dpi = 300, plot = last_plot())
```

###Word Cloud Twitter Tech Elite and Population###

Tech Elite
```{r}
#Preprocessing data
Twitter_Tech_corpus <- corpus(Twitter_Tech)
Twitter_Tech_dfm <- dfm(Twitter_Tech_corpus)

#tokenize the text and remove punctuation, numbers
Twitter_Tech_toks <- tokens(Twitter_Tech_corpus, remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, include_docvars = TRUE)

#convert everything to lower case
Twitter_Tech_toks <- tokens_tolower(Twitter_Tech_toks)

#remove stopwords
Twitter_Tech_toks <- tokens_remove(Twitter_Tech_toks, stopwords('en'))
Twitter_Tech_dfm <- dfm(Twitter_Tech_toks)

#most frequent 100 words in dfm
#topfeatures(Twitter_Tech_dfm, 100)

#remove individual letters
Twitter_Tech_dfm <- dfm(Twitter_Tech_toks, remove=c("?", "amp", "ðÿ", "itâ", "iâ", "re", "weâ", "donâ", "˜", "˜*", "que", "canâ", "thatâ", "youâ", "la", "ll", "yâ", "ðÿž", "en", "via", "ve", "de"))

#basic wordcloud with 50 words
#tiff(file="TechCloud50.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Twitter_Tech_dfm, min_size = 1, max_size = 5, min_count = 5, max_words = 50, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")))
#dev.off()

#basic wordcloud with 100 words
#tiff(file="TechCloud100.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Twitter_Tech_dfm, min_size = 1, max_size = 5, min_count = 5, max_words = 100, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")))
#dev.off()
```

Population 
```{r}
Pop_corpus <- corpus(Twitter_Pop)
Pop_dfm <- dfm(Pop_corpus)
Pop_toks <- tokens(Pop_corpus, remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, include_docvars = TRUE)
Pop_toks <- tokens_tolower(Pop_toks)
Pop_toks <- tokens_remove(Pop_toks, stopwords('en'))
Pop_dfm <- dfm(Pop_toks)

#most frequent 100 words in dfm
#topfeatures(Pop_dfm, 100)

Pop_dfm <- dfm(Pop_toks, remove=c("?", "amp", "ðÿ", "itâ", "iâ", "re", "weâ", "donâ", "˜", "˜*", "que", "canâ", "thatâ", "youâ", "la", "ll", "yâ", "ðÿž", "en", "via", "ve", "de"), verbose=TRUE)

#basic wordcloud with 50 words
#tiff(file="PopCloud50.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
popcloud50 <- textplot_wordcloud(Pop_dfm, min_size = 1, max_size = 5, max_words= 50, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")),)
#dev.off()

#basic wordcloud with 100 words
#tiff(file="PopCloud100.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
popcloud100 <- textplot_wordcloud(Pop_dfm, min_size = 1, max_size = 5, max_words= 100, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")),)
#dev.off()
```

###Merging Twitter Population and Tech Elite Datasets Including Dummy###

```{r}
Pop_matches <- Twitter_Pop[,1:2]
dummy <- c('Population')
Pop_matches <- data.frame(Pop_matches, dummy)

Tech_matches <- Twitter_Tech[,1:2]
dummy <- c('Tech')
Tech_matches <- data.frame(Tech_matches, dummy)
head(Tech_matches)

Tech_matches_a <- Tech_matches[,1:3]
Pop_matches_b <- Pop_matches[,1:3]

Twitter_both <- rbind(Tech_matches_a, Pop_matches_b)

#names(Twitter_both)

#write.csv(Twitter_both, file = "Twitter_both.csv")
```

###Dictionary Approach Twitter Data###

```{r}
#Preprocessing
Twitter_both_corpus<- corpus(Twitter_both, docid_field = "doc_id",
  text_field = "text", metacorpus = NULL, compress = FALSE)
Twitter_both_dfm <- Twitter_both_corpus %>%
  corpus_subset(dummy %in% c("Tech", "Population")) %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, groups = "dummy") %>%
  dfm(remove=c("?", "amp", "ðÿ", "itâ", "iâ", "re", "weâ", "donâ", "˜", "˜*", "que", "canâ", "thatâ", "youâ", "la", "ll", "yâ", "ðÿž", "en", "via", "ve", "de"), verbose=TRUE) %>%
  dfm_trim(min_termfreq = 3)

#Reading in LIWC Dictionary (needs to be purchased individually)
liwc2015 <- quanteda::dictionary(file = "C:/Users/Kohout Admin/Dropbox/Philanthropy Hilke/Final Data Analysis/Datasets/dict.yml")

#Creating Merit, Future and Democracy Dictionary
demo <- dictionary(list(demo_words = c("democ*", "demos", "republic", "self-government", "self-rule", "pure democracy", "home rule", "self-determination", "autonomy", "sovereignity", "sovranty", "popular", "republican", "self-governing", "self-ruling", "represent*", "libertarian", "nontotalitarian", "demonstrations", "rallies", "assembl*", "conferenc*", "congress*", "convention*", "convocation*", "council*", "gathering*", "march*", "protest*", "sit-down*", "sit-in*", "strike*", "counterdemonstration*", "counter-demonstration*", "counterprotest*", "counter-protest*", "counterrall*", "counter-rall*")))
merit <- dictionary(list(merit_words = c("merit*", "cardinal virtue", "distinct*", "excellen*", "grace", "value", "virtue", "advantage", "edge", "plus", "superiority", "account", "valuation", "worth", "assessment", "estimation", "evaluation", "great*", "perfect*", "consequence", "importan*", "significan*", "weight", "desirability", "deserve", "earn", "rate", "entitle", "qualify")))
future <- dictionary(list(future_words = c("futur*", "coming", "unborn", "approaching", "forthcoming", "imminent", "impending", "nearing", "oncoming", "pending", "upcoming", "after", "ensuing", "later", "posterior", "subsequent", "anticipated", "awaited", "expect*", "plan*", "predict*", "project*", "prospective", "eventual*", "final", "last", "ultereior", "ultimate", "by-and-by", "hereafter", "offing2", "tomorrow", "eventuality", "finality", "posterity", "fortune", "cirumstance", "destiny", "doom", "fate", "hap", "kismet", "lot", "portion", "futurities", "outlook", "prospect")))

#Combining all three into one dictionary
liwc2015plus <- dictionary(c(as.list(liwc2015), as.list(demo), as.list(merit), as.list(future)))

#running our advanced dictionary
liwcdfm <- dfm(Twitter_both_corpus, dictionary = liwc2015plus)

#add dictionary attributes as a new variable
Twitter_both$Function <- as.numeric(liwcdfm[,1])
Twitter_both$Pronoun <- as.numeric(liwcdfm[,2])
Twitter_both$Ppron <- as.numeric(liwcdfm[,3])
Twitter_both$I <- as.numeric(liwcdfm[,4])
Twitter_both$We <- as.numeric(liwcdfm[,5])
Twitter_both$You <- as.numeric(liwcdfm[,6])
Twitter_both$SheHe <- as.numeric(liwcdfm[,7])
Twitter_both$They <- as.numeric(liwcdfm[,8])
Twitter_both$Ipron <- as.numeric(liwcdfm[,9])
Twitter_both$Article <- as.numeric(liwcdfm[,10])
Twitter_both$Prep <- as.numeric(liwcdfm[,11])
Twitter_both$Auxverb <- as.numeric(liwcdfm[,12])
Twitter_both$Adverb <- as.numeric(liwcdfm[,13])
Twitter_both$Conj <- as.numeric(liwcdfm[,14])
Twitter_both$Negate <- as.numeric(liwcdfm[,15])
Twitter_both$Verb <- as.numeric(liwcdfm[,16])
Twitter_both$Adj <- as.numeric(liwcdfm[,17])
Twitter_both$Compare <- as.numeric(liwcdfm[,18])
Twitter_both$Interrog <- as.numeric(liwcdfm[,19])
Twitter_both$Number <- as.numeric(liwcdfm[,20])
Twitter_both$Quant <- as.numeric(liwcdfm[,21])
Twitter_both$Affect <- as.numeric(liwcdfm[,22])
Twitter_both$Posemo <- as.numeric(liwcdfm[,23])
Twitter_both$Negemo <- as.numeric(liwcdfm[,24])
Twitter_both$Anx <- as.numeric(liwcdfm[,25])
Twitter_both$Anger <- as.numeric(liwcdfm[,26])
Twitter_both$Sad <- as.numeric(liwcdfm[,27])
Twitter_both$Social <- as.numeric(liwcdfm[,28])
Twitter_both$Family <- as.numeric(liwcdfm[,29])
Twitter_both$Friend <- as.numeric(liwcdfm[,30])
Twitter_both$Female <- as.numeric(liwcdfm[,31])
Twitter_both$Male <- as.numeric(liwcdfm[,32])
Twitter_both$CogProc <- as.numeric(liwcdfm[,33])
Twitter_both$Insight <- as.numeric(liwcdfm[,34])
Twitter_both$Cause <- as.numeric(liwcdfm[,35])
Twitter_both$Discrep <- as.numeric(liwcdfm[,36])
Twitter_both$Tentat <- as.numeric(liwcdfm[,37])
Twitter_both$Certain <- as.numeric(liwcdfm[,38])
Twitter_both$Differ <- as.numeric(liwcdfm[,39])
Twitter_both$Percept <- as.numeric(liwcdfm[,40])
Twitter_both$See <- as.numeric(liwcdfm[,41])
Twitter_both$Hear <- as.numeric(liwcdfm[,42])
Twitter_both$Feel <- as.numeric(liwcdfm[,43])
Twitter_both$Bio <- as.numeric(liwcdfm[,44])
Twitter_both$Body <- as.numeric(liwcdfm[,45])
Twitter_both$Health <- as.numeric(liwcdfm[,46])
Twitter_both$Sexual <- as.numeric(liwcdfm[,47])
Twitter_both$Ingest <- as.numeric(liwcdfm[,48])
Twitter_both$Drives <- as.numeric(liwcdfm[,49])
Twitter_both$Affiliation <- as.numeric(liwcdfm[,50])
Twitter_both$Achieve <- as.numeric(liwcdfm[,51])
Twitter_both$Power <- as.numeric(liwcdfm[,52])
Twitter_both$Reward <- as.numeric(liwcdfm[,53])
Twitter_both$Risk <- as.numeric(liwcdfm[,54])
Twitter_both$FocusPast <- as.numeric(liwcdfm[,55])
Twitter_both$FocusPresent <- as.numeric(liwcdfm[,56])
Twitter_both$FocusFuture <- as.numeric(liwcdfm[,57])
Twitter_both$Relativ <- as.numeric(liwcdfm[,58])
Twitter_both$Motion <- as.numeric(liwcdfm[,59])
Twitter_both$Space <- as.numeric(liwcdfm[,60])
Twitter_both$Time <- as.numeric(liwcdfm[,61])
Twitter_both$Work <- as.numeric(liwcdfm[,62])
Twitter_both$Leisure <- as.numeric(liwcdfm[,63])
Twitter_both$Home <- as.numeric(liwcdfm[,64])
Twitter_both$Money <- as.numeric(liwcdfm[,65])
Twitter_both$Relig <- as.numeric(liwcdfm[,66])
Twitter_both$Death <- as.numeric(liwcdfm[,67])
Twitter_both$Informal <- as.numeric(liwcdfm[,68])
Twitter_both$Swear <- as.numeric(liwcdfm[,69])
Twitter_both$Netspeak <- as.numeric(liwcdfm[,70])
Twitter_both$Assent <- as.numeric(liwcdfm[,71])
Twitter_both$Nonflu <- as.numeric(liwcdfm[,72])
Twitter_both$Filler <- as.numeric(liwcdfm[,73])
Twitter_both$democracy <- as.numeric(liwcdfm[,74])
Twitter_both$merit <- as.numeric(liwcdfm[,75])
Twitter_both$future <- as.numeric(liwcdfm[,76])

#write.csv(Twitter_both, file = "Twitter_both.csv")
```


####Sentiment Analysis with LIWC for Twitter Population vs. Tech Elite and in different subgroups####

Sentiment in General
```{r}
#Aggregate Number of Positive and Negative Words in Tweets by group
aggregate(Twitter_both$Posemo, by=list(Category=Twitter_both$dummy), FUN=sum)
aggregate(Twitter_both$Negemo, by=list(Category=Twitter_both$dummy), FUN=sum)

#T-Test on Sentiments Among Future-Related Tweets
t.test(Twitter_both$Posemo  ~ Twitter_both$dummy, var.equal=TRUE)
t.test(Twitter_both$Negemo  ~ Twitter_both$dummy, var.equal=TRUE)
```

Sentiment in Tweets referring to Future-related words only
```{r}
#subsetting for future-related tweets
Twitter_both_future <- subset(Twitter_both, Twitter_both$future>=1)

#Aggregate Number of Positive and Negative Words in Future-related Tweets by group
aggregate(Twitter_both_future$Posemo, by=list(Category=Twitter_both_future$dummy), FUN=sum)
aggregate(Twitter_both_future$Negemo, by=list(Category=Twitter_both_future$dummy), FUN=sum)

#T-Test on Sentiments Among Future-Related Tweets
t.test(Twitter_both_future$Posemo  ~ Twitter_both_future$dummy, var.equal=TRUE)
t.test(Twitter_both_future$Negemo  ~ Twitter_both_future$dummy, var.equal=TRUE)
```

Sentiment in Tweets referring to Merit-related words only
```{r}
#subsetting for Merit-related Tweets
Twitter_both_merit <- subset(Twitter_both, Twitter_both$merit>=1)

#Aggregate Number of Positive and Negative Words in Merit-related Tweets by group
aggregate(Twitter_both_merit$Posemo, by=list(Category=Twitter_both_merit$dummy), FUN=sum)
aggregate(Twitter_both_merit$Negemo, by=list(Category=Twitter_both_merit$dummy), FUN=sum)

#T-Test on Sentiments Among Merit-Related Tweets
t.test(Twitter_both_merit$Posemo  ~ Twitter_both_merit$dummy, var.equal=TRUE)
t.test(Twitter_both_merit$Negemo  ~ Twitter_both_merit$dummy, var.equal=TRUE)
```

Achieve-related words only
```{r}
#Aggregate Number of Achieve-Related Tweets by Group
aggregate(Twitter_both$Achieve, by=list(Category=Twitter_both$dummy), FUN=sum)
t.test(Twitter_both$Achieve  ~ Twitter_both$dummy, var.equal=TRUE)

#subsetting for Achieve-Related Tweets
Twitter_both_achieve <- subset(Twitter_both, Twitter_both$Achieve>=1)

#Aggregate Number of Positive and Negative Words in Achieve-related Tweets by group
aggregate(Twitter_both_achieve$Posemo, by=list(Category=Twitter_both_achieve$dummy), FUN=sum)
aggregate(Twitter_both_achieve$Negemo, by=list(Category=Twitter_both_achieve$dummy), FUN=sum)

#T-Test on Sentiments Among Achieve-Related Tweets
t.test(Twitter_both_achieve$Posemo  ~ Twitter_both_achieve$dummy, var.equal=TRUE)
t.test(Twitter_both_achieve$Negemo  ~ Twitter_both_achieve$dummy, var.equal=TRUE)
```

####Keyness Statistics Twitter Tech Elite vs. Population####

Futur-related words
```{r}
#Aggregate Number of Merit-related Tweets by group
aggregate(Twitter_both$future, by=list(Category=Twitter_both$dummy), FUN=sum)

#preprocessing
Twitter_future_corpus<- corpus(Twitter_both_future, docid_field = "doc_id", text_field = "text", metacorpus = NULL, compress = FALSE)
Twitter_future_dfm <- Twitter_future_corpus %>%
  corpus_subset(dummy %in% c("Tech", "Population")) %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, remove_hyphens = TRUE, groups = "dummy") %>%
  dfm(remove=c("?", "amp", "ðÿ", "itâ", "iâ", "re", "weâ", "donâ", "˜", "˜*", "que", "canâ", "thatâ", "youâ", "la", "ll", "yâ", "ðÿž", "en", "via", "ve", "de"), verbose=TRUE) %>%
  dfm_trim(min_termfreq = 3)

#most frequent 100 words in dfm
#topfeatures(Twitter_future_dfm, 100)

#keyness statistics
#tiff(file="KeynessTwitterFuture.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
tstat_key <- textstat_keyness(Twitter_future_dfm, target = "Tech")
textplot_keyness(tstat_key, n= 12, margin = 0.2, show_legend = TRUE, labelsize = 4)
#dev.off()
```

Merit-related words
```{r}
#Aggregate Number of Merit-related Tweets by group
aggregate(Twitter_both$merit, by=list(Category=Twitter_both$dummy), FUN=sum)

#preprocessing
Twitter_merit_corpus<- corpus(Twitter_both_merit, docid_field = "doc_id", text_field = "text", metacorpus = NULL, compress = FALSE)
Twitter_merit_dfm <- Twitter_merit_corpus %>%
  corpus_subset(dummy %in% c("Tech", "Population")) %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, remove_hyphens = TRUE, groups = "dummy") %>%
  dfm(remove=c("?", "amp", "ðÿ", "itâ", "iâ", "re", "weâ", "donâ", "˜", "˜*", "que", "canâ", "thatâ", "youâ", "la", "ll", "yâ", "ðÿž", "en", "via", "ve", "de"), verbose=TRUE) %>%
  dfm_trim(min_termfreq = 3)

#most frequent 100 words in dfm
#topfeatures(Twitter_merit_dfm, 100)

#keyness statistics
#tiff(file="KeynessTwitterMerit.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
tstat_key <- textstat_keyness(Twitter_merit_dfm, target = "Tech")
textplot_keyness(tstat_key, n= 12, margin = 0.2, show_legend = TRUE, labelsize = 4)
#dev.off()
```

achieve-related words
```{r}
#Aggregate Number of Merit-related Tweets by group
aggregate(Twitter_both$Achieve, by=list(Category=Twitter_both$dummy), FUN=sum)

#preprocessing
Twitter_achieve_corpus<- corpus(Twitter_both_achieve, docid_field = "doc_id", text_field = "text", metacorpus = NULL, compress = FALSE)
Twitter_achieve_dfm <- Twitter_achieve_corpus %>%
  corpus_subset(dummy %in% c("Tech", "Population")) %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, remove_hyphens = TRUE, groups = "dummy") %>%
  dfm(remove=c("?", "amp", "ðÿ", "itâ", "iâ", "re", "weâ", "donâ", "˜", "˜*", "que", "canâ", "thatâ", "youâ", "la", "ll", "yâ", "ðÿž", "en", "via", "ve", "de"), verbose=TRUE) %>%
  dfm_trim(min_termfreq = 3)

#most frequent 100 words in dfm
#topfeatures(Twitter_achieve_dfm, 100)

#keyness statistics
#tiff(file="KeynessTwitterAchieve.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
tstat_key <- textstat_keyness(Twitter_achieve_dfm, target = "Tech")
textplot_keyness(tstat_key, n= 12, margin = 0.2, show_legend = TRUE, labelsize = 4)
#dev.off()
```

####Dictionary Approach for Pledge Letters####

```{r}
#Reading Pledge Letters
Pledge <- readtext(file="GPMS.xlsx", text_field="pledge_letter")

#Preprocessing 
Pledge_corpus<- corpus(Pledge, docid_field = "doc_id", text_field = "text", metacorpus = NULL, compress = FALSE)
Pledge_dfm <- Pledge_corpus %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, groups = "Techies") %>%
  dfm_trim(min_termfreq = 3)

#most frequent 100 words in dfm
#topfeatures(Pledge_dfm, 100)

#running the dictionary on the corpus
liwcdfm <- dfm(Pledge_corpus, dictionary = liwc2015plus)

#add dictionary attributes as a new variable
Pledge$Function <- as.numeric(liwcdfm[,1])
Pledge$Pronoun <- as.numeric(liwcdfm[,2])
Pledge$Ppron <- as.numeric(liwcdfm[,3])
Pledge$I <- as.numeric(liwcdfm[,4])
Pledge$We <- as.numeric(liwcdfm[,5])
Pledge$You <- as.numeric(liwcdfm[,6])
Pledge$SheHe <- as.numeric(liwcdfm[,7])
Pledge$They <- as.numeric(liwcdfm[,8])
Pledge$Ipron <- as.numeric(liwcdfm[,9])
Pledge$Article <- as.numeric(liwcdfm[,10])
Pledge$Prep <- as.numeric(liwcdfm[,11])
Pledge$Auxverb <- as.numeric(liwcdfm[,12])
Pledge$Adverb <- as.numeric(liwcdfm[,13])
Pledge$Conj <- as.numeric(liwcdfm[,14])
Pledge$Negate <- as.numeric(liwcdfm[,15])
Pledge$Verb <- as.numeric(liwcdfm[,16])
Pledge$Adj <- as.numeric(liwcdfm[,17])
Pledge$Compare <- as.numeric(liwcdfm[,18])
Pledge$Interrog <- as.numeric(liwcdfm[,19])
Pledge$Number <- as.numeric(liwcdfm[,20])
Pledge$Quant <- as.numeric(liwcdfm[,21])
Pledge$Affect <- as.numeric(liwcdfm[,22])
Pledge$Posemo <- as.numeric(liwcdfm[,23])
Pledge$Negemo <- as.numeric(liwcdfm[,24])
Pledge$Anx <- as.numeric(liwcdfm[,25])
Pledge$Anger <- as.numeric(liwcdfm[,26])
Pledge$Sad <- as.numeric(liwcdfm[,27])
Pledge$Social <- as.numeric(liwcdfm[,28])
Pledge$Family <- as.numeric(liwcdfm[,29])
Pledge$Friend <- as.numeric(liwcdfm[,30])
Pledge$Female <- as.numeric(liwcdfm[,31])
Pledge$Male <- as.numeric(liwcdfm[,32])
Pledge$CogProc <- as.numeric(liwcdfm[,33])
Pledge$Insight <- as.numeric(liwcdfm[,34])
Pledge$Cause <- as.numeric(liwcdfm[,35])
Pledge$Discrep <- as.numeric(liwcdfm[,36])
Pledge$Tentat <- as.numeric(liwcdfm[,37])
Pledge$Certain <- as.numeric(liwcdfm[,38])
Pledge$Differ <- as.numeric(liwcdfm[,39])
Pledge$Percept <- as.numeric(liwcdfm[,40])
Pledge$See <- as.numeric(liwcdfm[,41])
Pledge$Hear <- as.numeric(liwcdfm[,42])
Pledge$Feel <- as.numeric(liwcdfm[,43])
Pledge$Bio <- as.numeric(liwcdfm[,44])
Pledge$Body <- as.numeric(liwcdfm[,45])
Pledge$Health <- as.numeric(liwcdfm[,46])
Pledge$Sexual <- as.numeric(liwcdfm[,47])
Pledge$Ingest <- as.numeric(liwcdfm[,48])
Pledge$Drives <- as.numeric(liwcdfm[,49])
Pledge$Affiliation <- as.numeric(liwcdfm[,50])
Pledge$Achieve <- as.numeric(liwcdfm[,51])
Pledge$Power <- as.numeric(liwcdfm[,52])
Pledge$Reward <- as.numeric(liwcdfm[,53])
Pledge$Risk <- as.numeric(liwcdfm[,54])
Pledge$FocusPast <- as.numeric(liwcdfm[,55])
Pledge$FocusPresent <- as.numeric(liwcdfm[,56])
Pledge$FocusFuture <- as.numeric(liwcdfm[,57])
Pledge$Relativ <- as.numeric(liwcdfm[,58])
Pledge$Motion <- as.numeric(liwcdfm[,59])
Pledge$Space <- as.numeric(liwcdfm[,60])
Pledge$Time <- as.numeric(liwcdfm[,61])
Pledge$Work <- as.numeric(liwcdfm[,62])
Pledge$Leisure <- as.numeric(liwcdfm[,63])
Pledge$Home <- as.numeric(liwcdfm[,64])
Pledge$Money <- as.numeric(liwcdfm[,65])
Pledge$Relig <- as.numeric(liwcdfm[,66])
Pledge$Death <- as.numeric(liwcdfm[,67])
Pledge$Informal <- as.numeric(liwcdfm[,68])
Pledge$Swear <- as.numeric(liwcdfm[,69])
Pledge$Netspeak <- as.numeric(liwcdfm[,70])
Pledge$Assent <- as.numeric(liwcdfm[,71])
Pledge$Nonflu <- as.numeric(liwcdfm[,72])
Pledge$Filler <- as.numeric(liwcdfm[,73])
Pledge$democracy <- as.numeric(liwcdfm[,74])
Pledge$merit <- as.numeric(liwcdfm[,75])
Pledge$future <- as.numeric(liwcdfm[,76])

#write.csv(Pledge, file = "Pledge.csv")

#Aggregate Number of Positive and Negative Words in Pledge Statements
sum(Pledge$Posemo)
sum(Pledge$Negemo)

#Aggregate Number of Positive and Negative Words by Group
aggregate(Pledge$Posemo, by=list(Category=Pledge$Techies), FUN=sum)
aggregate(Pledge$Negemo, by=list(Category=Pledge$Techies), FUN=sum)

#T-Test on Sentiments By Group
t.test(Pledge$Posemo  ~ Pledge$Techies, var.equal=TRUE)
t.test(Pledge$Negemo  ~ Pledge$Techies, var.equal=TRUE)
```

####Normalize Pledge Letters by Text Length####

```{r}
#collapse into two documents
Pledge_dfm1 <- dfm(Pledge_corpus, groups = "Techies")

#turn word counts into proportions
Pledge_dfm1[1:2, 1:10]
Pledge_dfm1 <- dfm_weight(Pledge_dfm1, scheme = "prop")
Pledge_dfm1[1:2, 1:10]

#Proportional Positive sentiment
posemo.words <- liwc2015plus$Posemo
posemo_dic <- dictionary(list(posemo = posemo.words))
posemo_sent <- dfm_lookup(Pledge_dfm1, dictionary = posemo_dic)
posemo_sent
posemo_sent*100

#Proportional Negative Sentiment
negemo.words <- liwc2015plus$Negemo
negemo_dic <- dictionary(list(negemo = negemo.words))
negemo_sent <- dfm_lookup(Pledge_dfm1, dictionary = negemo_dic)
negemo_sent
negemo_sent*100

#Proportional Religion
religious.words <- liwc2015plus$Relig
religion_dic <- dictionary(list(religion = religious.words))
relig_sent <- dfm_lookup(Pledge_dfm1, dictionary = religion_dic)
relig_sent
relig_sent*100
```

####World Cloud Giving Pledge####

All Pledges (Letters)
```{r}
#basic wordcloud with 50 words
#tiff(file="PledgeCloud50.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Pledge_dfm, min_size = 1, max_size = 5, min_count = 5, max_words = 50, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")))
#dev.off()

#basic wordcloud with 100 words
#tiff(file="PledgeCloud100.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Pledge_dfm, min_size = 1, max_size = 5, min_count = 5, max_words = 100, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")))
#dev.off()
```

Tech Elite vs. No Tech Elite Comparison
```{r}
#preprocessing
Pledge_dfm2 <- Pledge_corpus %>%
  corpus_subset(Techies %in% c("Tech Elite", "No Tech Elite")) %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, remove_hyphens = TRUE, groups = "Techies") %>%
  dfm_trim(min_termfreq = 3)

#basic wordcloud with 50 words
#tiff(file="PledgeTechNoTechCloud.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Pledge_dfm2, comparison = TRUE, min_size = 1, max_size = 3, min_count = 5, max_words = 50, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")))
#dev.off()
```

Tech Elite Only
```{r}
Pledge_dfm3 <- Pledge_corpus %>%
  corpus_subset(Techies %in% c("Tech Elite")) %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, remove_hyphens = TRUE, groups = "Techies") %>%
  dfm_trim(min_termfreq = 3)

#tiff(file="PledgeTechCloud.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Pledge_dfm3, min_size = 1, max_size = 4, min_count = 5, max_words = 50, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")))
#dev.off()
```

No Tech Elite only
```{r}
Pledge_dfm4 <- Pledge_corpus %>%
  corpus_subset(Techies %in% c("No Tech Elite")) %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, remove_hyphens = TRUE, groups = "Techies") %>%
  dfm_trim(min_termfreq = 3)

#tiff(file="PledgeNoTechCloud.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Pledge_dfm4, min_size = 1, max_size = 5, min_count = 5, max_words = 50, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")))
#dev.off()
```

Keyness Statistics Giving Pledge Tech Elite vs No Tech Elite
```{r}
#keyness statistics
#tiff(file="KeynessPledgeTechNoTech.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
tstat_key <- textstat_keyness(Pledge_dfm2, target = "Tech Elite")
textplot_keyness(tstat_key, n= 12, margin = 0.2, show_legend = TRUE, labelsize = 4)
#dev.off()
```

####Dictionary Approach for Mission Statements####

```{r}
#Reading Website Mission Statements
Mission <- readtext(file="GPMS.xlsx", text_field="mission_statement")

#Preprocessing
Mission_corpus<- corpus(Mission, docid_field = "doc_id", text_field = "text", metacorpus = NULL, compress = FALSE)
Mission_dfm <- Mission_corpus %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, groups = "Techies") %>%
  dfm_trim(min_termfreq = 3)

#most frequent 100 words in dfm
#topfeatures(Mission_dfm, 100)

#running the dictionary on the corpus
liwcdfm <- dfm(Mission_corpus, dictionary = liwc2015plus)

#add dictionary attributes as a new variable
Mission$Function <- as.numeric(liwcdfm[,1])
Mission$Pronoun <- as.numeric(liwcdfm[,2])
Mission$Ppron <- as.numeric(liwcdfm[,3])
Mission$I <- as.numeric(liwcdfm[,4])
Mission$We <- as.numeric(liwcdfm[,5])
Mission$You <- as.numeric(liwcdfm[,6])
Mission$SheHe <- as.numeric(liwcdfm[,7])
Mission$They <- as.numeric(liwcdfm[,8])
Mission$Ipron <- as.numeric(liwcdfm[,9])
Mission$Article <- as.numeric(liwcdfm[,10])
Mission$Prep <- as.numeric(liwcdfm[,11])
Mission$Auxverb <- as.numeric(liwcdfm[,12])
Mission$Adverb <- as.numeric(liwcdfm[,13])
Mission$Conj <- as.numeric(liwcdfm[,14])
Mission$Negate <- as.numeric(liwcdfm[,15])
Mission$Verb <- as.numeric(liwcdfm[,16])
Mission$Adj <- as.numeric(liwcdfm[,17])
Mission$Compare <- as.numeric(liwcdfm[,18])
Mission$Interrog <- as.numeric(liwcdfm[,19])
Mission$Number <- as.numeric(liwcdfm[,20])
Mission$Quant <- as.numeric(liwcdfm[,21])
Mission$Affect <- as.numeric(liwcdfm[,22])
Mission$Posemo <- as.numeric(liwcdfm[,23])
Mission$Negemo <- as.numeric(liwcdfm[,24])
Mission$Anx <- as.numeric(liwcdfm[,25])
Mission$Anger <- as.numeric(liwcdfm[,26])
Mission$Sad <- as.numeric(liwcdfm[,27])
Mission$Social <- as.numeric(liwcdfm[,28])
Mission$Family <- as.numeric(liwcdfm[,29])
Mission$Friend <- as.numeric(liwcdfm[,30])
Mission$Female <- as.numeric(liwcdfm[,31])
Mission$Male <- as.numeric(liwcdfm[,32])
Mission$CogProc <- as.numeric(liwcdfm[,33])
Mission$Insight <- as.numeric(liwcdfm[,34])
Mission$Cause <- as.numeric(liwcdfm[,35])
Mission$Discrep <- as.numeric(liwcdfm[,36])
Mission$Tentat <- as.numeric(liwcdfm[,37])
Mission$Certain <- as.numeric(liwcdfm[,38])
Mission$Differ <- as.numeric(liwcdfm[,39])
Mission$Percept <- as.numeric(liwcdfm[,40])
Mission$See <- as.numeric(liwcdfm[,41])
Mission$Hear <- as.numeric(liwcdfm[,42])
Mission$Feel <- as.numeric(liwcdfm[,43])
Mission$Bio <- as.numeric(liwcdfm[,44])
Mission$Body <- as.numeric(liwcdfm[,45])
Mission$Health <- as.numeric(liwcdfm[,46])
Mission$Sexual <- as.numeric(liwcdfm[,47])
Mission$Ingest <- as.numeric(liwcdfm[,48])
Mission$Drives <- as.numeric(liwcdfm[,49])
Mission$Affiliation <- as.numeric(liwcdfm[,50])
Mission$Achieve <- as.numeric(liwcdfm[,51])
Mission$Power <- as.numeric(liwcdfm[,52])
Mission$Reward <- as.numeric(liwcdfm[,53])
Mission$Risk <- as.numeric(liwcdfm[,54])
Mission$FocusPast <- as.numeric(liwcdfm[,55])
Mission$FocusPresent <- as.numeric(liwcdfm[,56])
Mission$FocusFuture <- as.numeric(liwcdfm[,57])
Mission$Relativ <- as.numeric(liwcdfm[,58])
Mission$Motion <- as.numeric(liwcdfm[,59])
Mission$Space <- as.numeric(liwcdfm[,60])
Mission$Time <- as.numeric(liwcdfm[,61])
Mission$Work <- as.numeric(liwcdfm[,62])
Mission$Leisure <- as.numeric(liwcdfm[,63])
Mission$Home <- as.numeric(liwcdfm[,64])
Mission$Money <- as.numeric(liwcdfm[,65])
Mission$Relig <- as.numeric(liwcdfm[,66])
Mission$Death <- as.numeric(liwcdfm[,67])
Mission$Informal <- as.numeric(liwcdfm[,68])
Mission$Swear <- as.numeric(liwcdfm[,69])
Mission$Netspeak <- as.numeric(liwcdfm[,70])
Mission$Assent <- as.numeric(liwcdfm[,71])
Mission$Nonflu <- as.numeric(liwcdfm[,72])
Mission$Filler <- as.numeric(liwcdfm[,73])
Mission$democracy <- as.numeric(liwcdfm[,74])
Mission$merit <- as.numeric(liwcdfm[,75])
Mission$future <- as.numeric(liwcdfm[,76])

#Aggregate Number of Positive and Negative Words in Mission Statements
sum(Mission$Posemo)
sum(Mission$Negemo)

#write.csv(Mission, file = "Mission.csv")
```

####Cohort Analysis of Mission Statements###
```{r}
#Generating Cohort Variable
Mission$age <- as.numeric(as.character(Mission$age))

Mission$cohort <- Mission$age
Mission$cohort <- ifelse((Mission$age>=0 & Mission$age<=44) , 'internet',Mission$cohort)
Mission$cohort <- ifelse((Mission$age>44 & Mission$age<=65) , 'software',Mission$cohort)
Mission$cohort <- ifelse((Mission$age>65), 'hardware',Mission$cohort)

Mission$cohort<-as.factor(Mission$cohort)
summary(Mission$cohort)

#sentiment by cohort
aggregate(Mission$Posemo, by=list(Category=Mission$cohort), FUN=sum)

#transform categorical cohort into numeric
Mission$cohort_num <- as.numeric(Mission$cohort)
#Mission$cohort_num
#hardware=1, internet=2, software=3

#subset into internet&software and internet&hardware
int_soft <- subset(Mission, Mission$cohort_num >= 2)
int_hard <- subset(Mission, Mission$cohort_num <= 2)

#T-Test on Positive Sentiments By Cohort Groups
t.test(int_soft$Posemo  ~ int_soft$cohort, var.equal=TRUE)
t.test(int_hard$Posemo  ~ int_hard$cohort, var.equal=TRUE)

#T-Test on Religious Words by Cohort Groups
t.test(int_soft$Relig  ~ int_soft$cohort, var.equal=TRUE)
t.test(int_hard$Relig  ~ int_hard$cohort, var.equal=TRUE)
```

####Word Cloud Mission Statements####

All Statements
```{r}
#basic wordcloud with 50 words
#tiff(file="MissionCloud50.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Mission_dfm, min_size = 1, max_size = 5, min_count = 5, max_words = 50, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")))
#dev.off()

#basic wordcloud with 100 words
#tiff(file="MissionCloud100.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Mission_dfm, min_size = 1, max_size = 5, min_count = 5, max_words = 100, rotation = 0.1, color = rev(RColorBrewer::brewer.pal(10, "RdBu")))
#dev.off()
```

By Cohort
```{r}
#preprocessing
Mission_corpus<- corpus(Mission, docid_field = "doc_id", text_field = "text", metacorpus = NULL, compress = FALSE)
Mission_dfm1 <- Mission_corpus %>%
  corpus_subset(cohort %in% c("software", "internet", "hardware")) %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, remove_hyphens = TRUE, groups = "cohort") %>%
  dfm_trim(min_termfreq = 3)

#basic wordcloud with 50 words
#tiff(file="MissionCloudCohort.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
textplot_wordcloud(Mission_dfm1, comparison = TRUE, min_size = 1, max_size = 4, min_count = 5, max_words = 50, rotation = 0.1, color = c("blue", "red", "dark green"))
#dev.off()
```

####Democracy-Related Words####

Sentiment Analysis
```{r}
#subsetting for Democracy-Related Tweets
Twitter_both_democracy <- subset(Twitter_both, Twitter_both$democracy>=1)

#Aggregate Number of Democracy Words by Group
aggregate(Twitter_both$democracy, by=list(Category=Twitter_both$dummy), FUN=sum)

#Aggregate Number of Positive and Negative Words in Democracy-Related Tweets by Group
aggregate(Twitter_both_democracy$Posemo, by=list(Category=Twitter_both_democracy$dummy), FUN=sum)
aggregate(Twitter_both_democracy$Negemo, by=list(Category=Twitter_both_democracy$dummy), FUN=sum)

#T-Test on Sentiments Among Democracy-Related Tweets
t.test(Twitter_both_democracy$Posemo  ~ Twitter_both_democracy$dummy, var.equal=TRUE)
t.test(Twitter_both_democracy$Negemo  ~ Twitter_both_democracy$dummy, var.equal=TRUE)
```

Keyness Statistics
```{r}
#preprocessing
Twitter_democracy_corpus<- corpus(Twitter_both_democracy, docid_field = "doc_id", text_field = "text", metacorpus = NULL, compress = FALSE)
Twitter_democracy_dfm <- Twitter_democracy_corpus %>%
  corpus_subset(dummy %in% c("Tech", "Population")) %>%
  dfm(remove = stopwords("english"), remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE, remove_twitter = TRUE, remove_url = TRUE, remove_hyphens = TRUE, groups = "dummy") %>%
  dfm(remove=c("?", "amp", "ðÿ", "itâ", "iâ", "re", "weâ", "donâ", "˜", "˜*", "que", "canâ", "thatâ", "youâ", "la", "ll", "yâ", "ðÿž", "en", "via", "ve", "de"), verbose=TRUE) %>%
  dfm_trim(min_termfreq = 3)

#keyness statistics
#tiff(file="KeynessDemocracy.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
tstat_key <- textstat_keyness(Twitter_democracy_dfm, target = "Tech")
textplot_keyness(tstat_key, n= 12, margin = 0.2, show_legend = TRUE, labelsize = 4)
#dev.off()
```

####Power, Money and Democracy Correlation####

T-Tests Tech Elite vs. Pop
```{r}
t.test(Twitter_both$Power  ~ Twitter_both$dummy, var.equal=TRUE)
t.test(Twitter_both$Money  ~ Twitter_both$dummy, var.equal=TRUE)

#Among Democracy-Related Words
t.test(Twitter_both_democracy$Power  ~ Twitter_both_democracy$dummy, var.equal=TRUE)
t.test(Twitter_both_democracy$Money  ~ Twitter_both_democracy$dummy, var.equal=TRUE)
```

Subset Twitter Data into Elite vs. Population
```{r}
#transform categorical dummy into numeric 
Twitter_both$dummy_num <- as.numeric(Twitter_both$dummy)
Twitter_both$dummy_num

#subset into tech elite vs population
elite <- subset(Twitter_both, Twitter_both$dummy_num==1)
pop <- subset(Twitter_both, Twitter_both$dummy_num==2)
```

Elite 
```{r}
#Money & Power
cor.test(elite$Money, elite$Power)

#Power & Democracy
cor.test(elite$Power, elite$democracy)

#Money & Democracy
cor.test(elite$Money, elite$democracy)
```

Population
```{r}
#Money & Power
cor.test(pop$Money, pop$Power)

#Power & Democracy
cor.test(pop$Power, pop$democracy)

#Money & Democracy
cor.test(pop$Money, pop$democracy)
```

####Prediction####

##Logistic Regression

```{r}
#install.packages("aod")
library(aod)

Twitter_both$dummy_num <- factor(Twitter_both$dummy_num)
logit_model <- glm(dummy_num ~ future + merit + Achieve + Relig + democracy + Power + Money + Posemo + Negemo, data = Twitter_both, family = "binomial")

summary(logit_model)

#Calculate R2
with(summary(logit_model), 1 - deviance/null.deviance)
```

##Naive Bayes Classifyer (https://tutorials.quanteda.io/machine-learning/nb/)
```{r}
#install.packages("caret", dependencies = TRUE)
#install.libraries("quanteda.textmodels")
library(caret)
library(quanteda.textmodels)

#generate random sample of 80% of the documents
set.seed(300)
id_train <- sample(1:99654, 79723, replace = FALSE)
head(id_train, 10)

#create docvar with ID
docvars(Twitter_both_corpus, "id_numeric") <- 1:ndoc(Twitter_both_corpus)

#get training set
dfmat_training <- corpus_subset(Twitter_both_corpus, id_numeric %in% id_train) %>%
    dfm(stem = TRUE)

#get test set (documents not in id_train)
dfmat_test <- corpus_subset(Twitter_both_corpus, !id_numeric %in% id_train) %>%
    dfm(stem = TRUE)

#train the naive Bayes classifier
tmod_nb_dummy <- textmodel_nb(dfmat_training, docvars(dfmat_training, "dummy"))
summary(tmod_nb_dummy)

#saveRDS(tmod_nb_dummy, file = "C:/Users/Kohout Admin/Dropbox/Philanthropy Hilke/FINAL Data Analysis/Models/naive.rds")
#tmod_nb_dummy <- readRDS(file = "C:/Users/Kohout Admin/Dropbox/Philanthropy Hilke/FINAL Data Analysis/Models/naive.rds")

dfmat_matched <- dfm_match(dfmat_test, features = featnames(dfmat_training))

#classification performance
actual_class <- docvars(dfmat_matched, "dummy")

predicted_class <- predict(tmod_nb_dummy, newdata = dfmat_matched)

tab_class <- table(actual_class, predicted_class)
tab_class

#confusion matrix
confusionMatrix(tab_class, mode = "everything")

#########ROCR CURVE#########
#install.packages("ROCR")
library(ROCR)

predvec <- ifelse(predicted_class=="Tech", 1, 0)
realvec <- ifelse(actual_class=="Tech", 1, 0)
 
pred_r <- prediction(predvec, realvec)
perf_r <- performance(pred_r, "tpr", "fpr")

#tiff(file="Naive.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
plot(perf_r)
#dev.off()

###AUC####
auc.perf = performance(pred_r, measure = "auc")
auc.perf@y.values
```

###Support Vector Machine Classification with 50.000 Sample Size
```{r}
#install.packages("e1071")
#install.packages("SparseM")
#install.packages("doParallel)
#install.packages("foreach")
#install.packages("tictoc")
library(e1071)
library(SparseM)
library(doParallel)
library(foreach)
library(tictoc)

####LINEAR KERNEL####

set.seed(123)
tweets50000 <- Twitter_both[sample(nrow(Twitter_both), 50000),]

# clean text and create DFM
tweets50000$text <- gsub('@[0-9_A-Za-z]+', '@', tweets50000$text)
twcorpus <- corpus(tweets50000$text)
twdfm <- dfm(twcorpus, remove=stopwords("english"), remove_url=TRUE, 
             ngrams=1:2, verbose=TRUE)
twdfm <- dfm_trim(twdfm, min_docfreq = 5, verbose=TRUE)

training <- sample(1:nrow(tweets50000), floor(.80 * nrow(tweets50000)))
test <- (1:nrow(tweets50000))[1:nrow(tweets50000) %in% training == FALSE]

registerDoParallel(cores = 4)
tic()
fit <- tune(svm, train.x=twdfm[training,], 
            train.y=factor(tweets50000$dummy[training]),
            kernel="linear",
            ranges=list(cost=c(0.1, 1, 10, 50, 100)),
            decision.values = TRUE)
summary(fit)
toc()

# best model
bestmodel <- fit$best.model
summary(bestmodel)

#saveRDS(bestmodel, file = "svm_50000_linear.rds")
#bestmodel <- readRDS(file = "svm_50000_linear.rds")

###
# evaluating performance
pred1 <- predict(bestmodel, twdfm[test,])

## function to compute accuracy
accuracy <- function(ypred, y){
	tab <- table(ypred, y)
	return(sum(diag(tab))/sum(tab))
}
# function to compute precision
precision <- function(ypred, y){
	tab <- table(ypred, y)
	return((tab[2,2])/(tab[2,1]+tab[2,2]))
}
# function to compute recall
recall <- function(ypred, y){
	tab <- table(ypred, y)
	return(tab[2,2]/(tab[1,2]+tab[2,2]))
}

# confusion matrix
table(pred1, tweets50000$dummy[test])
# performance metrics
accuracy(pred1, tweets50000$dummy[test])
precision(pred1, tweets50000$dummy[test])
recall(pred1, tweets50000$dummy[test])

###ROC CURVE####
library(ROCR)
fitted <- attributes(predict(bestmodel, twdfm[training,], decision.values = TRUE))$decision.values
pred_r1 <- prediction(fitted, as.vector(tweets50000$dummy[training]))
pred_r1
perf_r1 <- performance(pred_r1, "tpr", "fpr" )
perf_r1

#tiff(file="SVM_50000_lin.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
plot(perf_r1)
#dev.off()

###AUC####
auc.perf1 = performance(pred_r1, measure = "auc")
auc.perf1@y.values
```

###SVM Polynomial Kernel

```{r}
# clean text and create DFM
set.seed(123)
tweets50000 <- Twitter_both[sample(nrow(Twitter_both), 50000),]

# clean text and create DFM
tweets50000$text <- gsub('@[0-9_A-Za-z]+', '@', tweets50000$text)
twcorpus <- corpus(tweets50000$text)
twdfm <- dfm(twcorpus, remove=stopwords("english"), remove_url=TRUE, 
             ngrams=1:2, verbose=TRUE)
twdfm <- dfm_trim(twdfm, min_docfreq = 5, verbose=TRUE)

training <- sample(1:nrow(tweets50000), floor(.80 * nrow(tweets50000)))
test <- (1:nrow(tweets50000))[1:nrow(tweets50000) %in% training == FALSE]

registerDoParallel(cores = 4)
tic()
fit_pol <- tune(svm, train.x=twdfm[training,], 
                train.y=factor(tweets50000$dummy[training]), 
                kernel="polynomial", 
                probability = TRUE,
                ranges=list(cost=c(0,1, 1, 10, 50, 100)))

summary(fit_pol)
toc()

# best model
bestmodel1 <- fit_pol$best.model
summary(bestmodel1)

#saveRDS(bestmodel1, file = "svm_50000_poly.rds")
#bestmodel1 <- readRDS(file = "svm_50000_poly.rds")

###
# evaluating performance
pred1 <- predict(bestmodel1, twdfm[test,])

# confusion matrix
table(pred1, tweets50000$dummy[test])
# performance metrics
accuracy(pred1, tweets50000$dummy[test])
precision(pred1, tweets50000$dummy[test])
recall(pred1, tweets50000$dummy[test])

###ROC CURVE####
library(ROCR)

fitted1 <- attributes(predict(bestmodel1, twdfm[training,], decision.values = TRUE))$decision.values
pred_r2 <- prediction(fitted1, as.vector(tweets50000$dummy[training]))
pred_r2
perf_r2 <- performance(pred_r2, "tpr", "fpr" )
perf_r2

#tiff(file="SVM_50000_pol.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
plot(perf_r2)
#dev.off()
```

####RANDOM FOREST MODEL

```{r}
#install.packages("randomForest")
#install.packages("ranger")
#install.packages("caTools")
library(randomForest)
library(ranger)
library(caTools)
library(MLmetrics)

####RANGER##### 

set.seed(123)

# clean text and create DFM
Twitter_both$text <- gsub('@[0-9_A-Za-z]+', '@', Twitter_both$text)
twcorpus <- corpus(Twitter_both$text)
twdfm <- dfm(twcorpus, remove=stopwords("english"), remove_url=TRUE, 
             ngrams=1:2, verbose=TRUE)
twdfm <- dfm_trim(twdfm, min_docfreq = 5, verbose=TRUE)

training <- sample(1:nrow(Twitter_both), floor(.80 * nrow(Twitter_both)))
test <- (1:nrow(Twitter_both))[1:nrow(Twitter_both) %in% training == FALSE]

X <- as.matrix(dfm_trim(twdfm, min_docfreq = 50,
                        max_docfreq=0.80*nrow(twdfm), verbose=TRUE))

#First time around we will not use a resampling method.
trctrl <- trainControl(method = "none")

#The fourth model will be the ranger model from the caTools package. Where we specify default parameters
rf_mod <- train(x=X[training,],
                y=factor(Twitter_both$dummy[training]),
                method = "ranger",
                trControl = trctrl,
                tuneGrid = data.frame(mtry = floor(sqrt(dim(twdfm)[2])),
                                      splitrule = "gini",
                                      min.node.size = 1))


#saveRDS(rf_mod, "final_tree.rds")
#rf_mod <- readRDS(file = "final_tree.rds")

#Prediction on test data
rf_pred <- predict(rf_mod, X[test,])

# confusion matrix
confusionMatrix(rf_pred, Twitter_both$dummy[test])

####ROC CURVE####
library(ROCR)

predvec <- ifelse(rf_pred=="Tech", 1, 0)

pred_r3 <- prediction(predvec, as.vector(Twitter_both$dummy[test]))
perf_r3 <- performance(pred_r3, "tpr", "fpr")

#tiff(file="forest.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
plot(perf_r3)
#dev.off()

####Calculating AUC#####
auc.perf3 = performance(pred_r3, measure = "auc")
auc.perf3@y.values
```

###ROC CURVE FOR ALL###
```{r}
#tiff(file="together.tiff", height = 12, width = 17, units = 'cm', compression = "lzw", res = 300)
plot(perf_r, col="red")
plot(perf_r1, col = "blue", add = TRUE)
plot(perf_r3, col="black", add = TRUE)
legend("bottomright", legend=c("Naive Bayes", "linear SVM", "Random Forest"), col=c("red", "blue", "black"), lty=1, cex=0.8)
abline(a=0, b= 1, col="grey")
#dev.off()
```

#####ANOVA#####
```{r}
accuracy_df <- c(0.8384, 0.8052, 0.7911)
sensitivity_df <- c(0.7944, 0.7726, 0.7998)
specificity_df <- c(0.9002, 0.8663, 0.7824)

y <- c(accuracy_df, sensitivity_df, specificity_df)

n <- rep(1, 2, 3)
n

group <- rep(1:3, n)
group

data_a = data.frame(y=y, group=factor(group))
fit_a = lm(y ~ group, data_a)
anova(fit_a)
```
