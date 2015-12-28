---
#Do Workplace Freedoms Translate into Increased Job Satisfaction?
###Schinria Islam
###December 21, 2015
---

###Packages
```{r}
stopifnot(require("psych"))
stopifnot(require("QMSS"))
stopifnot(require("VGAM"))
library(psych)
library(QMSS)
library(VGAM)

```


```{r}
#uploading data
rawData <- read.csv("GSS2006.csv") #

vars <- c("wrkstat","satjob1","wkfreedm","slfmangd","partteam","setthngs","wksub","lotofsay","wkdecide","wrkhome","famwkoff","mustwork","chngtme","condemnd","jobsecok","race","workfor","joblose","wkdecide","sex","polviews","age","rincom06")

intro.vars <- c("wrkindp","flexhrs")

d <- rawData[ ,vars] #
intro.d <- rawData[ ,intro.vars]

dim(d)
dim(intro.d)
```

```{r}
#Intro paragraph recodes/dummies
intro.d$wrkindp = 6-intro.d$wrkindp # reverse coded to reflect increasing importance
intro.d$flexhrs = 6-intro.d$flexhrs # reverse coded to reflect increasing importance
intro.d$impwrkindp = ifelse((intro.d$wrkindp>3),1,0) #
intro.d$impflexhrs = ifelse((intro.d$flexhrs>3),1,0) #

tabwrkindp <- table(intro.d$impwrkindp) #
tabflexhrs <- table(intro.d$impflexhrs) #
prop.table(tabwrkindp) #percent of those who think independence at work is important
prop.table(tabflexhrs) #percent of who who think flexible working hours are important
```

####Description of Data Set and Variables
```{r}
#independence in working style
d$wkownduty <- 5-d$wkfreedm #
d$wkfreedm <- d$wkownduty
d <- d[,1:23]
colnames(d)[3] <- "wkownduty"
d$slfmangd = ifelse((d$slfmangd==1),1,0) 
d$partteam = ifelse((d$partteam==2),1,0) 
d$wksub = ifelse((d$wksub==2),1,0)
d$lotofsay = 6-d$lotofsay 
d$wkdecide = 5-d$wkdecide 
d$setthngs = 5-d$setthngs 
```

```{r}
#schedule flexibility / work flexibility
d$famwkoff = 5-d$famwkoff #
d$mustwork = ifelse((d$mustwork==2),1,0) #
d$chngtme = 5-d$chngtme #
d$condemnd = 5-d$condemnd #
```


```{r}
#dependent variable
d$satjob1 = 5-d$satjob1 #

#subset data to only those working full-time, exclusing missings
d.no.na <- na.omit(d)
d.sub <- subset(d.no.na, d.no.na$wrkstat==1)
```

####Descriptive Statistics

```{r}
summary(d.sub)
describe(d.sub)
```

```{r}
#distribution of responses for dependent variable

d.sub$jobsat <- factor(d.sub$satjob1, levels = 1:4, labels = c("Not at all Satisfied","Not too Satisfied", "Somewhat Satisfied","Very Satisfied"), ordered=TRUE)

#seeing if our dataset is now all complete and ready for analysis
ok <- complete.cases(d.sub)
sum(!ok) 
  
dim(d.sub)
Tab(d.sub$jobsat)
```

```{r}
#initial ordinal logit regression
vglm.jobsat <- vglm(jobsat ~ wkownduty + slfmangd + partteam + wksub + lotofsay + wkdecide + setthngs + wrkhome + famwkoff + mustwork + chngtme + condemnd, data = d.sub, family = propodds)

summary(vglm.jobsat)

coef.jobsat <- coef(summary(vglm.jobsat))
coef.jobsat <- data.frame(coef.jobsat)
coef.jobsat$odds.ratio <- exp(coef.jobsat[, "Estimate"])
coef.jobsat
```

```{r}
#some more ordinal logistic regression, specific to Very Satisfied and Very Satisfied/Somewhat Satisfied
d.sub$jobsat_4 <- d.sub$jobsat=="Very Satisfied"
d.sub$jobsat_34 <- with(d.sub, jobsat=="Very Satisfied" | jobsat=="Somewhat Satisfied")

#1
logit.34_vs_12 <- glm(jobsat_34 ~ wkownduty + slfmangd + partteam + wksub + lotofsay + wkdecide + setthngs + wrkhome + famwkoff + mustwork + chngtme + condemnd, data = d.sub, family=binomial)

summary(logit.34_vs_12)
```

```{r}
#2
logit.4_vs_123 <- glm(jobsat_4 ~ wkownduty + slfmangd + partteam + wksub + lotofsay + wkdecide + setthngs + wrkhome + famwkoff + mustwork + chngtme + condemnd, data = d.sub, family=binomial)

summary(logit.4_vs_123)
```

```{r}
#comparing all models
coeff.table <- cbind("Logit (3,4 vs. 1,2)" = coef(logit.34_vs_12), "Logit (4 vs. 1,2,3)" = coef(logit.4_vs_123), "Ordinal Logit" = coef(vglm.jobsat)[-c(1,2)])

print(coeff.table, digits=3)
```


```{r}
#alpha for work independence 
vars.wf1 <- c("wkownduty", "lotofsay", "wkdecide", "setthngs")
sub.wf1 <- d.sub[ ,vars.wf1]
cor(sub.wf1)
summary(alpha(sub.wf1))

#alpha for schedule flexibility

vars.wf2 <- c("famwkoff", "condemnd")
sub.wf2 <- d.sub[ ,vars.wf2]
View(sub.wf2)
cor(sub.wf2)
summary(alpha(sub.wf2), check.keys=TRUE)

#creating scale for work independence, WF1
keys <- c(1, 1, 1, 1)
d.sub$wkindp <- rowMeans(scale(sub.wf1))
View(d.sub)
```

```{r}
#descriptive statistics of scale
describe(sub.wf1)
```

```{r}
#ordinal logit regression with scale

vglm.jobsat.scale <- vglm(jobsat ~ wkindp + wrkhome + famwkoff + condemnd, data = d.sub, family = propodds)

summary(vglm.jobsat.scale)

coef.jobsat.scale <- coef(summary(vglm.jobsat.scale))
coef.jobsat.scale <- data.frame(coef.jobsat.scale)
coef.jobsat.scale$odds.ratio <- exp(coef.jobsat.scale[, "Estimate"])
coef.jobsat.scale
```

```{r}
#some more ordinal logistic regression, specific to Very Satisfied and Very Satisfied/Somewhat Satisfied

#1 with scale
logit.34_vs_12_scale <- glm(jobsat_34 ~ wkindp + wrkhome + famwkoff + condemnd, data = d.sub, family=binomial)

summary(logit.34_vs_12_scale)
```

```{r}
#2 with scale
logit.4_vs_123_scale <- glm(jobsat_4 ~ wkindp + wrkhome + famwkoff + condemnd, data = d.sub, family=binomial)

summary(logit.4_vs_123_scale)
```

```{r}
#comparing all models
coeff.table.scale <- cbind("Logit (3,4 vs. 1,2) w/ Scale" = coef(logit.34_vs_12), "Logit (4 vs. 1,2,3) w/ Scale" = coef(logit.4_vs_123), "Ordinal Logit w/ Scale" = coef(vglm.jobsat.scale)[-c(1,2)])

print(coeff.table.scale, digits=3)
```


```{r}
#control variables
View(d.sub)

d.sub$sex <- ifelse((d.sub$sex==1),0,1)

#again
describe(d.sub)

```

```{r}
#ordinal regression with control variables
vglm.jobsat.sl.cont <- vglm(jobsat ~ wkindp + wrkhome + famwkoff + condemnd + age + sex + rincom06, data = d.sub, family = propodds)

summary(vglm.jobsat.sl.cont)

coef.jobsat.sl.cont <- coef(summary(vglm.jobsat.sl.cont))
coef.jobsat.sl.cont <- data.frame(coef.jobsat.sl.cont)
coef.jobsat.sl.cont$odds.ratio <- exp(coef.jobsat.sl.cont[, "Estimate"])
coef.jobsat.sl.cont

#looking at income alone
vglm.income <- vglm(jobsat ~ rincom06, data = d.sub, family = propodds)
summary(vglm.income)
```

```{r}
vglm.jobsat.sl.cont2 <- vglm(jobsat ~ wkindp + wrkhome + famwkoff + condemnd + age, data = d.sub, family = propodds)

summary(vglm.jobsat.sl.cont2)

coef.jobsat.sl.cont2 <- coef(summary(vglm.jobsat.sl.cont2))
coef.jobsat.sl.cont2 <- data.frame(coef.jobsat.sl.cont2)
coef.jobsat.sl.cont2$odds.ratio <- exp(coef.jobsat.sl.cont2[, "Estimate"])
coef.jobsat.sl.cont2
```

```{r}
#interactions

vglm.jobsat.int <- vglm(jobsat ~ wkindp + wrkhome + famwkoff + condemnd + age + age*condemnd, data = d.sub, family = propodds)

summary(vglm.jobsat.int)

coef.jobsat.int <- coef(summary(vglm.jobsat.int))
coef.jobsat.int <- data.frame(coef.jobsat.int)
coef.jobsat.int$odds.ratio <- exp(coef.jobsat.int[, "Estimate"])
coef.jobsat.int
```

```{r}
#final ordinal logistic regression, specific to Very Satisfied and Very Satisfied/Somewhat Satisfied

#1 with scale, control, and interaction
logit.34_vs_12_int <- glm(jobsat_34 ~ wkindp + wrkhome + famwkoff + condemnd + age + age*condemnd, data = d.sub, family=binomial)

summary(logit.34_vs_12_int)
```

```{r}
#2 with scale, control, and interaction
logit.4_vs_123_int <- glm(jobsat_4 ~ wkindp + wrkhome + famwkoff + condemnd + age + age*condemnd, data = d.sub, family=binomial)

summary(logit.4_vs_123_int)
```

```{r}
#comparing all models
coeff.table.int <- cbind("Logit (3,4 vs. 1,2) w/ Scale" = coef(logit.34_vs_12_int), "Logit (4 vs. 1,2,3) w/ Scale" = coef(logit.4_vs_123_int), "Ordinal Logit w/ Scale" = coef(vglm.jobsat.int)[-c(1,2)])

print(coeff.table.int, digits=3)
```

```{r}
#test for parallel slopes
vglm.jobsat.int2 <- vglm(jobsat ~ wkindp + wrkhome + famwkoff + condemnd + age + age*condemnd, data = d.sub, family = cumulative(reverse=T))

summary(vglm.jobsat.int2)

propOddsTest(vglm.jobsat.int, vglm.jobsat.int2)

```

