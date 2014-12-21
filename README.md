# Getting and Cleaning Data Course Project

The Course Project uses data from: https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 

The codebook for this project is available at:
https://github.com/arunksri/Getting-and-Cleaning-Data/blob/master/CodeBook.md

## Assignment

 You should create one R script called run_analysis.R that does the following. 

 1. Merges the training and the test sets to create one data set.
 2. Extracts only the measurements on the mean and standard deviation for each measurement. 
 3. Uses descriptive activity names to name the activities in the data set
 4. Appropriately labels the data set with descriptive variable names. 
 5. Creates a second, independent tidy data set with the average of each
    variable for each activity and each subject.

###1. Merges the training and the test sets to create one data set

Download and unzip data files

if(!file.info("UCI HAR Dataset")$isdir) {
        download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip",
                      destfile="./UCI HAR Dataset.zip")
        unzip("./UCI HAR Dataset.zip")
}

Read test and train data with column names extracted from features

features <- read.table("./UCI HAR Dataset/features.txt")
test.data <- read.table("./UCI HAR Dataset/test/X_test.txt",col.names=features[,2],check.names=FALSE)
train.data <- read.table("./UCI HAR Dataset/train/X_train.txt",col.names=features[,2],check.names=FALSE)

Read test and train labels and subject with appropriate column names

test.label <- read.table("./UCI HAR Dataset/test/y_test.txt",col.names=c("Activity_Code"))
test.subject <- read.table("./UCI HAR Dataset/test/subject_test.txt",col.names=c("Subject"))
train.label <- read.table("./UCI HAR Dataset/train/y_train.txt",col.names=c("Activity_Code"))
train.subject <- read.table("./UCI HAR Dataset/train/subject_train.txt",col.names=c("Subject"))

###2. Extracts only the measurements on the mean and standard deviation for each measurement

Subset test and train data with only the required columns

cols.required <- grep("(mean|std)\\(",features[,2])
test.data.required <- test.data[,cols.required]
train.data.required <- train.data[,cols.required]

###3. Uses descriptive activity names to name the activities in the data set

Merge test and train data along with Subject and Activity COde

test.final <- cbind(test.label,test.subject,test.data.required)
test.final[,"Type"] <- c("Test")
train.final <- cbind(train.label,train.subject,train.data.required)
train.final[,"Type"] <- c("Train")
data.final <- rbind(test.final,train.final)

###4. Appropriately labels the data set with descriptive variable names

Add Activity Label description to the dataset

activity.label <- read.table("./UCI HAR Dataset/activity_labels.txt",col.names=c("Activity_Code","Activity_Desc"))
mergedData <- merge(activity.label,data.final,by=c("Activity_Code"))

###5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable 
###for each activity and each subject

Reshape and cast final data to obtain the tidy data set

library(reshape2)
id_labels = c("Activity_Code","Activity_Desc","Subject","Type")
data_labels = setdiff(colnames(mergedData), id_labels)
dataMelt = melt(mergedData, id = id_labels, measure.vars = data_labels)
tidyData <- dcast(dataMelt,Activity_Desc+Subject~variable,mean)

Write tidy dataset and codebook to a new file 

write.table(tidyData,file="./UCI HAR Tidy Dataset.txt",row.names=FALSE,sep="\t",quote=FALSE)
write.table(paste("* ", names(tidyData), sep=""), file="./CodeBook.md", quote=FALSE,
            row.names=FALSE, col.names=FALSE, sep="\t")
