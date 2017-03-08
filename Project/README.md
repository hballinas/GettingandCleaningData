#Project for Getting and Cleaning Data, Coursera - Johns Hopkins University
#Author: Horacio Ballinas
#Date: 3/8/2017

The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.

One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:

http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

Here are the data for the project:

https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

##You should create one R script called run_analysis.R that does the following.

Merges the training and the test sets to create one data set. 
Extracts only the measurements on the mean and standard deviation for each measurement. 
Uses descriptive activity names to name the activities in the data set. 
Appropriately labels the data set with descriptive activity names. 
Creates a second, independent tidy data set with the average of each variable for each activity and each subject. 

Good luck! Steps to reproduce this project

##Open the R script run_analysis.r using a text editor.

Once you download or clone the files... 
###FIRST - Change the parameter of the setwd function on run_analysis.r call to the working directory/folder. 
Remember you set your working directory every time you open RStudio. On this working directory you download run_analysis.Rmd as well. When I wrote the file I used Windows and an open source application to unzip the compressed file called Zip-7. I migrated everything to a Mac and used the unzip function loaded into the Mac OS.

###SECOND - Run the R script run_analysis.r. 
It calls the R Markdown file, run_analysis.Rmd, which contains the code. 
At the end there's a chunk indicating how to create the Codebook.md file.  Please advise that creating the .md file takes several minutes.

##Outputs produced

Tidy dataset file DatasetHumanActivityRecognitionUsingSmartphones.txt (tab-delimited text) 
Codebook file (codebook.md, codebook.Rmd is RStudio Markdown file)
