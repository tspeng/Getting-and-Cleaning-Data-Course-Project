---
title: Getting and Cleaning Data Course Project
output: html_document
---
##Download and unzip the dataset
```{r,echo=TRUE}
fileurl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileurl,"Dataset.zip")
unzip("Dataset.zip", exdir = "Dataset")
```

##1.Merges the training and the test sets to create one data set
### Reads the training and test sets
```{r,echo=TRUE}
X_train <- read.table("./Dataset/UCI HAR Dataset/train/X_train.txt")  
y_train <- read.table("./Dataset/UCI HAR Dataset/train/y_train.txt")  
subject_train <- read.table("./Dataset/UCI HAR Dataset/train/subject_train.txt")
X_test <- read.table("./Dataset/UCI HAR Dataset/test/X_test.txt")
y_test <- read.table("./Dataset/UCI HAR Dataset/test/y_test.txt")
subject_test <- read.table("./Dataset/UCI HAR Dataset/test/subject_test.txt")

folddir <- "./Dataset/UCI HAR Dataset/train/Inertial Signals/"
filename <-c("body_acc_x","body_acc_y","body_acc_z","body_gyro_x","body_gyro_y","body_gyro_z","total_acc_x","total_acc_y","total_acc_z")
for (i in 1:length(filename)) {
    assign(paste0(filename[i],"_train"), read.table(paste0(folddir,filename[i],"_train",".txt")))
}
folddir <- "./Dataset/UCI HAR Dataset/test/Inertial Signals/"
for (i in 1:length(filename)) {
    assign(paste0(filename[i],"_test"), read.table(paste0(folddir,filename[i],"_test",".txt")))
}
```

### Merges the sets to create one data set
```{r,echo=TRUE}
X <- rbind(X_train,X_test)
y <- rbind(y_train,y_test)
subject <- rbind(subject_train,subject_test)

body_acc_x <- rbind(body_acc_x_train,body_acc_x_test)
body_acc_y <- rbind(body_acc_y_train,body_acc_y_test)
body_acc_z <- rbind(body_acc_z_train,body_acc_z_test)
body_gyro_x <- rbind(body_gyro_x_train,body_gyro_x_test)
body_gyro_y <- rbind(body_gyro_y_train,body_gyro_y_test)
body_gyro_z <- rbind(body_gyro_z_train,body_gyro_z_test)
total_acc_x <- rbind(total_acc_x_train,total_acc_x_test)
total_acc_y <- rbind(total_acc_y_train,total_acc_y_test)
total_acc_z <- rbind(total_acc_z_train,total_acc_z_test)
```

##2.Extracts only the measurements on the mean and standard deviation for each measurement
```{r,echo=TRUE}
features <-  read.table("./Dataset/UCI HAR Dataset/features.txt")
features$V2 <- as.character(features$V2)
index<- grepl("mean",features$V2) | grepl("std",features$V2)   #Find the index for mean and std variable
extact_data <- X[,index]          #Extract the data
```

##3.Uses descriptive activity names to name the activities in the data set
```{r,echo=TRUE}
labels <-  read.table("./Dataset/UCI HAR Dataset/activity_labels.txt")
y$V1 <- as.factor(y$V1)
levels(y$V1) <- labels$V2 
```

##4.Appropriately labels the data set with descriptive variable names
```{r,echo=TRUE}
features <-  read.table("./Dataset/UCI HAR Dataset/features.txt")
features$V2 <- as.character(features$V2)
var_names <-gsub("\\()","",features$V2)  #Creates variable names using features
names(X) <- var_names             #Assign the variable names to X
names(y) <- c("activities")       #Assign the variable names to y
names(subject) <- c("identifier") #Assign the variable names to subject

win_names <- vector()            #Create the window names
for (i in 1:ncol(body_acc_x)) {
    win_names[i] <- paste0("window-",as.character(i))
}

names(body_acc_x) <- win_names    #Assign the variable names for body_acc_x, which apply the following ariables
names(body_acc_y) <- win_names
names(body_acc_z) <- win_names
names(body_gyro_x) <- win_names
names(body_gyro_y) <- win_names
names(body_gyro_z) <- win_names
names(total_acc_x) <- win_names
names(total_acc_y) <- win_names
names(total_acc_z) <- win_names
```

##5.From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
```{r,echo=TRUE}
ave_activity <- aggregate(X,list(y$activities),mean)       #Average of each variable for each activity 
ave_subject <-  aggregate(X,list(subject$identifier),mean) #Average of each variable for each subject
```