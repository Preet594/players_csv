# players_csv

library(tidyverse)
library(repr)
library(tidymodels)
library(ggplot2)
options(repr.matrix.max.rows = 6)

players <- read_csv("/home/jovyan/work/players_csv/players.csv")
