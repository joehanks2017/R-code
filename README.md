# R-code
#Final Project#
library(caret)
library(tidyverse)
A = read_csv('bank-full.csv')
str(A)

#plot each feature#
ggplot(data = A, mapping = aes(x = age, y = campaign, color = y)) +
  geom_point()

ggplot(data = A, mapping =aes(x = y, y = previous)) +
  geom_boxplot()

#Heat map#
A_y_marital = A %>% group_by(y, marital) %>% summarise(count = n())
ggplot(data = A_y_marital, mapping = aes(x = y, y = marital)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

A_y_job = A %>% group_by(y, job) %>% summarise(count = n())
ggplot(data = A_y_job, mapping = aes(x = y, y = job)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

A_y_education = A %>% group_by(y, education) %>% summarise(count = n())
ggplot(data = A_y_education, mapping = aes(x = y, y = education)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

A_y_default = A %>% group_by(y, default) %>% summarise(count = n())
ggplot(data = A_y_default, mapping = aes(x = y, y = default)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

A_y_housing = A %>% group_by(y, housing) %>% summarise(count = n())
ggplot(data = A_y_housing, mapping = aes(x = y, y = housing)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

A_y_loan = A %>% group_by(y, loan) %>% summarise(count = n())
ggplot(data = A_y_loan, mapping = aes(x = y, y = loan)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

A_y_contact = A %>% group_by(y, contact) %>% summarise(count = n())
ggplot(data = A_y_contact, mapping = aes(x = y, y = contact)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

A_y_month = A %>% group_by(y, month) %>% summarise(count = n())
ggplot(data = A_y_month, mapping = aes(x = y, y = month)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

A_y_poutcome = A %>% group_by(y, poutcome) %>% summarise(count = n())
ggplot(data = A_y_poutcome, mapping = aes(x = y, y = poutcome)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

#Select features#
A = mutate(A, y=as.factor(A$y)) %>%
  mutate(default=as.factor(A$default)) %>%
  mutate(education=as.factor(A$education)) %>%
  mutate(contact=as.factor(A$contact)) %>%
  mutate(housing=as.factor(A$housing)) %>%
  mutate(loan=as.factor(A$loan)) %>%
  mutate(marital=as.factor(A$marital)) %>%
  mutate(poutcome=as.factor(A$poutcome)) %>%
  mutate(job=as.factor(A$job))
A = select(A, y, duration, education, contact, default, housing, loan, marital, poutcome, job, campaign)
head(A)
str(A)

#Split Test/Train#
set.seed(777)
trainIndex = createDataPartition(A$y, p = 0.8, list = FALSE, times = 1)
ATrain = A[ trainIndex,]
ATest = A[-trainIndex,]

#Center and scale Data#
scaler = preProcess(ATrain, method = c("center", "scale"))
ATrain = predict(scaler, ATrain)
ATest = predict(scaler, ATest)
head(ATrain)

#Backward feature selection#
#1st Round#
#no marital#
knnModel = train(y ~ education+contact+duration+housing+loan+job+poutcome+campaign+default, 
                 data = ATrain, method = "knn", trControl=trainControl(method='none'), tuneGrid=data.frame(k=20))
ATestPrediction = predict(knnModel, ATest)
confusionMatrix(ATestPrediction, ATest$y)
#Plots of real and predicted#
pred_ATest = mutate(ATest, pred_y = ATestPrediction)
pred_real_1 = pred_ATest %>% group_by(y, pred_y) %>% summarise(count = n())
ggplot(data = pred_real_1, mapping = aes(x = y, y = pred_y)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

#2nd Round#
#no default#
knnModel = train(y ~ education+contact+duration+housing+loan+job+poutcome+campaign, 
                 data = ATrain, method = "knn", trControl=trainControl(method='none'), tuneGrid=data.frame(k=20))
ATestPrediction = predict(knnModel, ATest)
confusionMatrix(ATestPrediction, ATest$y)
#Plots of real and predicted#
pred_ATest = mutate(ATest, pred_y = ATestPrediction)
pred_real_1 = pred_ATest %>% group_by(y, pred_y) %>% summarise(count = n())
ggplot(data = pred_real_1, mapping = aes(x = y, y = pred_y)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")

#3rd Round#
#no contact#
knnModel = train(y ~ education+job+duration+housing+loan+poutcome+campaign, 
                 data = ATrain, method = "knn", trControl=trainControl(method='none'), tuneGrid=data.frame(k=20))
ATestPrediction = predict(knnModel, ATest)
confusionMatrix(ATestPrediction, ATest$y)
#Plots of real and predicted#
pred_ATest = mutate(ATest, pred_y = ATestPrediction)
pred_real_1 = pred_ATest %>% group_by(y, pred_y) %>% summarise(count = n())
ggplot(data = pred_real_1, mapping = aes(x = y, y = pred_y)) +
  geom_tile(mapping = aes(fill = count), color = "white") +
  scale_fill_gradient(low = "white", high ="steelblue")




#LR model building(Better Method)
library(glmnet)
library(mlbench)
library(glmnetUtils)
lr = glmnet(y ~ ., data = ATrain, family = "binomial")
prediction = predict(lr, ATest, type = "class", s = 0.01)
confusionMatrix(prediction, ATest$y)


#Build Decision Tree#
library(rpart)
library(rpart.plot)

#Entropy#
tree = rpart(y ~ education+contact+duration+housing+loan+job+poutcome+campaign, 
             data =ATrain, method = "class", parms = list(split = "information"))
printcp(tree)
#GINI Index#
tree = rpart(y ~ education+contact+duration+housing+loan+job+poutcome+campaign, 
             data =ATrain, method = "class", parms = list(split = "gini"))
printcp(tree)

#Plot the tree#
opar = par(no.readonly = T)
par(mfrow=c(1,2))
rpart.plot(tree,branch=1, type=4,fallen.leaves=T,cex=0.8, sub = "CART(gini)")
par(opar)

#Accuracy#
ATestPrediction = predict(tree, ATest, type = "class")
confusionMatrix(ATestPrediction, ATest$y)

