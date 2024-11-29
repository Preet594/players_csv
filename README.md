
# Final Project Report: Using Age and Level of Experience to Predict Individual Play Time on PlaiCraft Server

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

##To wrangle and clean our data we selected only the columns we will be using for our data analysis, filtered the played hours column so that our data set only includes observations from people that have played on the server, and mutated the experience column from a character variable to a numeric variable with Beginner = 1, Amateur = 2, Regular = 3, Pro = 4, and Veteran = 5.
select_data <- players |> 
    filter(played_hours > 0) |>
    select(played_hours, age, experience)
select_data

clean_data <- select_data |>
  filter(played_hours > 0) |>  
  mutate(experience = ifelse(experience == "Beginner", 1,
                              ifelse(experience == "Amateur", 2,
                              ifelse(experience == "Regular", 3,
                              ifelse(experience == "Pro", 4,
                              ifelse(experience == "Veteran", 5, NA))))))

clean_data

age_plot <- clean_data |>
                ggplot(aes(x=age, y=played_hours))+
                geom_bar(stat='identity')+
                xlab("Age of Player (in years)")+
                ylab("Total Time Spent Playing (in hours)") +
                 theme(text= element_text(size=12))
          
age_plot

experience_plot <- clean_data |>
                    ggplot(aes(x=experience, y=played_hours))+
                    geom_bar(stat='identity')+
                    xlab("Experience Level of Player(Beginner-Veteran)") +
                    ylab("Total Time Spent Playing (in hours)") +
                    theme(text= element_text(size=12))

experience_plot  

clean_data_split <- initial_split(clean_data, prop = 0.75, strata = played_hours)
clean_data_training <- training(clean_data_split)
clean_data_testing <- testing(clean_data_split)

clean_data_recipe<-recipe(played_hours ~ experience + age, data = clean_data_training ) |>
  step_scale(all_predictors()) |>
  step_center(all_predictors())

clean_data_spec <- nearest_neighbor(weight_func = "rectangular",
                              neighbors = tune()) |>
  set_engine("kknn") |>
  set_mode("regression")

clean_data_vfold <- vfold_cv(clean_data_training , v = 5, strata = played_hours)

clean_data_wkflw <- workflow() |>
  add_recipe(clean_data_recipe) |>
  add_model(clean_data_spec)
clean_data_wkflw

gridvals <- tibble(neighbors = seq(from = 1, to = 200, by = 1))

clean_data_results <- clean_data_wkflw |>
  tune_grid(resamples = clean_data_vfold, grid = gridvals) |>
  collect_metrics() |>
  filter(.metric == "rmse")
clean_data_results

clean_data_min <- clean_data_results |>
  filter(mean == min(mean))
clean_data_min

## Discussion

- summarize what you found
- discuss whether this is what you expected to find?
- discuss what impact could such findings have?
- discuss what future questions could this lead to?

