# players_csv

library(tidyverse)
library(repr)
library(tidymodels)
library(ggplot2)
source("cleanup.R")
options(repr.matrix.max.rows = 6)

players <- read_csv("/home/jovyan/work/players_csv/players.csv")
select_data <- players |> 
    filter(played_hours > 0) |>
    select(played_hours, age, experience)
select_data

clean_data <- mutate(select_data, 
    as_factor(experience))
clean_data
