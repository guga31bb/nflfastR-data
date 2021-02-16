# nflfastR-data
NFL play-by-play data scraped from the [`nflfastR` package](https://github.com/mrcaseb/nflfastR) going back to 1999. Each season contains both regular season and postseason data, with `game_type` or `week` denoting which.

Data are stored in the data folder, available as either compressed csv (.csv.gz) .rds, or .parquet.

___

## Load data using R

### nflfastR

The easiest way to load the play-by-play data in R is a new function in nflfastR v4.0. So after running

```r
install.packages("nflfastR")
```

all you need to do to load a bunch of seasons is

```r
# define which seasons shall be loaded
seasons <- 2018:2020
pbp <- nflfastR::load_pbp(seasons)
```

### Without nflfastR

If you don't want to use the above nflfastR function you can download the binary .rds format. The following example shows how to load the seasons 2018 to 2020 (binded into a single dataframe).

```R
# define which seasons shall be loaded
seasons <- 2018:2020
pbp <- purrr::map_dfr(seasons, function(x) {
  con <- url(glue::glue("https://raw.githubusercontent.com/guga31bb/nflfastR-data",
                        "/master/data/play_by_play_{x}.rds"))
  dat <- readRDS(con)
  close(con)
  dat
})

```

However, if you want to load the compressed csv data run this:
```R
# define which seasons shall be loaded
seasons <- 2018:2020
pbp <- purrr::map_df(seasons, function(x) {
  readr::read_csv(
    glue::glue("https://raw.githubusercontent.com/guga31bb/nflfastR-data/master/data/play_by_play_{x}.csv.gz")
  )
})
```

Or you can read .parquet like this:
```R
# define which seasons shall be loaded
seasons <- 2018:2020
pbp <- purrr::map_dfr(seasons, function(x) {
  download.file(glue::glue("https://raw.githubusercontent.com/guga31bb/nflfastR-data/master/data/play_by_play_{x}.parquet"), "tmp.parquet")
  df <- arrow::read_parquet("tmp.parquet")
  return(df)
}
)
```

___

## Load data using Python

If you are using Python you can load the compressed csv data. The following example written by [Deryck](https://twitter.com/Deryck_SG) (thanks a lot!) loads the seasons 2017 to 2019 (binded into a single pandas dataframe) as well as rosters (from 2000 to latest season):
```Python
import pandas as pd 

#Enter desired years of data
YEARS = [2019,2018,2017]

data = pd.DataFrame()

for i in YEARS:  
    #low_memory=False eliminates a warning
    i_data = pd.read_csv('https://github.com/guga31bb/nflfastR-data/blob/master/data/' \
                         'play_by_play_' + str(i) + '.csv.gz?raw=True',
                         compression='gzip', low_memory=False)

    #sort=True eliminates a warning and alphabetically sorts columns
    data = data.append(i_data, sort=True)

#Give each row a unique index
data.reset_index(drop=True, inplace=True)

```
