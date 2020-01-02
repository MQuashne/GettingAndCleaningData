#Load in libraries
library(dplyr)
library(tidyr)
library(readr)

#----Merges the training and the test sets to create one data set----#

#Read datasets into tbl_df class using dplyr
subject_test<-read_table("./UCI HAR Dataset/test/subject_test.txt", col_names=FALSE) #no column names included in file
X_test<-read_table("./UCI HAR Dataset/test/X_test.txt", col_names=FALSE) #no column names included in file
Y_test<-read_table("./UCI HAR Dataset/test/Y_test.txt", col_names=FALSE) #no column names included in file
subject_train<-read_table("./UCI HAR Dataset/train/subject_train.txt", col_names=FALSE) #no column names included in file
X_train<-read_table("./UCI HAR Dataset/train/X_train.txt", col_names=FALSE) #no column names included in file
Y_train<-read_table("./UCI HAR Dataset/train/Y_train.txt", col_names=FALSE) #no column names included in file
activity_labels<-read_table("./UCI HAR Dataset/activity_labels.txt", col_names=FALSE) #no column names included in file
features<-read_table("./UCI HAR Dataset/features.txt", col_names=FALSE) #no column names included in file

#Add the activity (Y) as a column to each X dataset (now because merge will reorder the dataset)
colnames(Y_test)<-"Activity"
colnames(Y_train)<-"Activity"
X_test<-tbl_df(cbind(Y_test,X_test))
X_train<-tbl_df(cbind(Y_train,X_train))

#Add the subject number as a column to each X dataset (now because merge will reorder the dataset)
colnames(subject_test)<-"Subject"
colnames(subject_train)<-"Subject"
X_test<-tbl_df(cbind(subject_test,X_test))
X_train<-tbl_df(cbind(subject_train,X_train))

#Add the obervation type (Test/Train) as a column to each X dataset (now because merge will reorder the dataset)
X_test<-mutate(X_test,Observation_Type = "Test")
X_train<-mutate(X_train,Observation_Type = "Train")

#Merge the datasets
X_set<-tbl_df(merge(X_test,X_train,all=TRUE))

#Add column names for each variable
colnames(X_set)<-c("Subject","Activity",unlist(features[,1]),"Observation_Type")

#Convert Subject, Activity, Observation Type to factors
X_set<-mutate(X_set,Subject=as.factor(Subject),Activity=as.factor(Activity), Observation_Type=as.factor(Observation_Type))
levels(X_set$Activity)<-unlist(activity_labels[,2]) #Replace ativity numbers with names from activity_labels.txt

#////Dataset Merged////#

#----Extracts only the measurements on the mean and standard deviation for each measurement.----#
X_agg<-select(X_set,Subject,Activity,contains("mean()"),contains("std()"))

#----From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.----#
Summary_Set<-group_by(X_agg,Subject,Activity) %>% summarize_all(funs(mean))
write.csv(Summary_Set,"Dataset.csv")