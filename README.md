#Description
The readme file decribes the steps taking to tidy and produce a subset for data for the project.

This files assume a certain file structure. Ensure you are in the working directory that has the extracte files.
this is the directory "..\UCI HAR Dataset" . This folder has the "Test" and "Train" folder

setwd("xx\\UCI HAR Dataset") here xx is the full path missing for the working directory.

#Packages used
library(dplyr)
library(tidyr)
library(sqldf)

please run the install.packages("dplyr")  and install.packages("tidyr") and also for sqldf ,if  not they are not in your library list.
load the libary so it is available for use in the application
library(dplyr)

#Read In Text Files
## Activity & Feature Files
activity_labels <- read.table("./activity_labels.txt",header= FALSE,sep="", colClasses = "character")
feature <- read.table("./features.txt", header= FALSE,sep="", colClasses = "character")

##Read Trainining files
x_train <- read.table("./train/x_train.txt", header= FALSE,sep="")
y_train <-  read.table("./train/y_train.txt", header= FALSE,sep="")
subject_train <-  read.table("./train/subject_train.txt", header= FALSE,sep="")

##Read Test Files
	x_test <- read.table("./test/x_test.txt", header= FALSE,sep="")
	y_test <- read.table("./test/y_test.txt", header= FALSE,sep="")
	subject_test <-read.table("./test/subject_test.txt", header= FALSE,sep="")

#Column Names	
##4. Appropriately labels the data set with descriptive variable names.  
	Doing this here as it is easier to relabel before merging all datassets

Give columns names to the activity label as this will be used in the final result. 
names(activity_labels)<- c("id","activity_description")
The feature file contains the names to the x datasets for both training and test.
	names(x_train) <-feature[,2]
	names(x_test) <-feature[,2]

The activity file contains the names for the y label. The y columns are now renamed using the data from the file
	names(y_train)<- c("activity")
	names(y_test)<- c("activity")

Rename the subject training data column subject
	names(subject_train)<- c("subject")
	names(subject_test)<- c("subject")


#Merging data

First the training data is bound together using the cbind and then the test data is bound using cbind.
	all_train <-cbind(x_train,y_train,subject_train)
	all_test <- cbind(x_test,y_test,subject_test)


Now the columns are all in place we can merge the training and test data using rbind.

	all_data<- rbind(all_train,all_test)

### Extract only the measurements on the mean and standard deviation for each measurement.

We now have all the data, we want to only extract the mean and standard deviation for all activities.
so we are looking for title where the mean() or std()

Get a list of all the mean column names

	meancolnames <- grep("mean",feature$V2,value = TRUE)

Get the list of all the std column names

	stdcolnames<- grep("std",feature$V2,value = TRUE)

create a character vetor of the two character lists

	colnames <-c("subject","activity",meancolnames,stdcolnames)
 
	meand_std<- subset(all_data, select =colnames)


###3. Uses descriptive activity names to name the activities in the data set
	mergedata <- merge(meand_std,activity_labels,by.x="activity",by.y="id",all=TRUE)


Get a data set and remove the acitvity ie 1,2,3... we now have the acitivity
description

	all_data2 <- select (mergedata, -activity)


### 5 From the data set in step 4, create a second, independent tidy data set
### with the average of each variable for each activity and each subject.
	
	Final result will be activity,subject,variable, avg()

Every column contains a different variable
The features are measures a bit like grades and should be in a column called features
we use the gather funcion in the tydr package


	res<-gather(all_data2,feature,"mean_std",-subject,-activity_description)

Ensure the mean_std column is  numeric.

	  res$mean_std <- as.numeric(res$mean_std)
	  
	  sqldf() # open a connection
	  
	  summ<-sqldf("select subject,activity_description,feature,avg(mean_std) from res  group by subject,activity_description,feature ")


##write final data to a file called finaldata.txt

	write.table(summ,file="finaldata.txt",sep=" ",row.names= FALSE)

##Removed unwanted variables

	  rm(meand_std)
	  
	  rm(all_data)
	  
	  rm(mergedata)
	  
	  rm(all_data2)
	  
	  rm(all_train)
	  
	  rm(all_test)
	  
	  rm(x_test)
	  
	  rm(x_train)
	  
	  rm(y_test)
	  
	  rm(y_train)
	  
	  rm(subject_test)
	  
	  rm(subject_train)
	  
	  rm(activity_labels)
	  
	  rm(feature)
	  
	  rm(res)