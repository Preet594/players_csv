# Final Project Report

By Prisha Budhiraja, Yiming Chen, Preet Dhillon,  Jaime Fraser, and Stephanie Tamkee

## Introduction

The data for this project is collected from a UBC research project. A MineCraft server called PlaiCraft was set up where players actions are recorded as they navigate through the world. This data consists of two files: players.csv and sessions.csv. The question we want to answer is, "How do age and level of experience affect an individual's playing time on the PlaiCraft server?" To answer this, question, only the players.csv dataset will be used because it contains data about the players' age, experience level, and playing time.

### The Dataset: players.csv
This players.csv file contains a list of all the unique players, including the following data about each player:
 - experience (identifies as Beginner, Amateur, Regular, Pro, or Veteran)
 - subscribe (whether or not they have subscribed to PlaiCraft)
 - hashedEmail (personal identifier)
 - played_hours (number of hours played on the server)
 - name (name of player)
 - gender
 - age
"individualId" and "organizationName" (logical data type) are also column headings in the data set, however, there is no data contained in these columns. Excluding these variables, there are 7 variables (Note: "hashedEmail" and "name" are unique to each individual and are not dependent variables) and 196 observations.


The following table describes each variable by its name, type of variable, any issues observed in the data, and any other potential issues:
   
|     Name     |    Type   |  Visible Issues  |  Potential Issues  |
| ------------ | --------- | ---------------- | ------------------ |
|  experience  | character | N/A              | It is unclear what each category represents in terms of experience |
|  subscribe   | logical   | N/A              | N/A                |
| hashedEamil  | character | Value is not readable              | Not usable as predictor                |
| played_hours | double    | Many values are 0, indicating many players have not played | N/A |
|     name     | character | N/A| Not usable as predictor                |
|    gender    | character | N/A              | Gender is not binary; if this data is to be used, this must be considered |
|      age     | double    | N/A              | The idea that age cannot go to infinity may be considered depending on the method used in this project |

Note: N/A indicates that no issue is observed from looking at data set, however, more issues may be detected or resolved later on.

## Methods & Results

library(tidyverse)
library(repr)
library(tidymodels)
library(ggplot2)
options(repr.matrix.max.rows = 6)

url <- "https://drive.google.com/uc?id=1Mw9vW0hjTJwRWx0bDXiSpYsO3gKogaPz"
players <- read_csv(url)
players

select_data <- players |> 
    filter(played_hours > 0) |>
    select(played_hours, age, experience)
select_data

clean_data <- mutate(select_data, 
    experience = as.numeric(experience))
clean_data
