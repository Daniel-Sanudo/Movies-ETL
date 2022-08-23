# Movies-ETL

Create a SQL database that will be used in a hackathon by extracting, transforming and loading the data from 3 different sources. These 3 files contain information about the each movie's title, director, cast, budget, box office and more details which are relevant to analyze a common trend for their box office success or failure. 

## Overview of Project

![Ratings_DB](/Resources/ratings_query.png)

The Ratings table consists of 26024289 entries with the following column information:

![Ratings_DF_Colums](/Resources/ratings_query_columns.png)

These dataset can be used to obtain both understand what each user prefers since we have their user ID, the movie ID and the rating. It could also be used to obtain the common perception of a movie according to the ratings given to it.

![Movies_DB](/Resources/movies_query.png)

The movies dataset has 6052 film entries, each of these entries contains information for the following categories:

![Movies_DB_Columns](/Resources/movies_query_columns.png)

This dataset has specific informatino for each film, ranging from their IMDB and Kaggle ID, to the cast, staff that work on the film, and how profitable it was.

## Analysis Process

The ETL process was done by first creating dataframes out of the source csv files(kaggle and movielens files) and json file (wikipedia) which were provided to us. Since the wikipedia data also contained information about tv series, we first had to filter them out by creating a series that contained only the data for films.

After excluding the TV series information, each movie entry had its alternate title columns removed since they were not relevant for the purpose of our database and would only cause the process to be slower when iterating through more columns; this was done with the following code:

~~~~
    for key in ['Also known as','Arabic','Cantonese','Chinese','French',
                'Hangul','Hebrew','Hepburn','Japanese','Literally',
                'Mandarin','McCune-Reischauer','Original title','Polish',
                'Revised Romanization','Romanized','Russian',
                'Simplified','Traditional','Yiddish']:
        if key in movie:
            alt_titles[key] = movie[key]
            movie.pop(key)
    if len(alt_titles) > 0:
        movie['alt_titles'] = alt_titles
~~~~

More details can be found in the clean_movie function. This funciton also changed the column names into a more standarized format.

The duplicate entries were removed using the imdb_id column and the dataframe.drop_duplicates method.

Afterwards, columns that had more than 90% missing information were dropped since the amount of information that would be filled could alter the dataset as well as it would be uncertain if it were extrapolated.

Using regular expressions, the budget and box office information was cleaned and transformed from a string value to a numeric value that could be more easily used; release date and runtime used a different regular expression form to achieve the same result.

A merge on the wikipedia and kaggle data was done after cleaning the datasets; after comparing the data, it was noticeable that there was some information missing between both sources so the following decisions were taken:

| Wiki | Kaggle | Resolution |
|----|----|----|
| title_wiki | title_kaggle | Drop wikipedia |
|running_time | runtime | Keep Kaggle; fill in zeros with Wikipedia data. |
|budget_wiki | budget_kaggle | Keep Kaggle; fill in zeros with Wikipedia data. |
|box_office | revenue | Keep Kaggle; fill in zeros with Wikipedia data. |
|release_date_wiki | release_date_kaggle | Drop wikipedia |
|Language  |  original_language | Drop wikipedia |
|Production company(s) | production_companies | Drop wikipedia |

The kaggle data was fileed with the wikipedia data using the following function:

~~~~
    def fill_missing_kaggle_data(df, kaggle_column, wiki_column):
        df.loc[:,kaggle_column] = df.apply(
            lambda row: row[wiki_column] 
            if row[kaggle_column] == 0 
            else row[kaggle_column], axis=1
        )
        df.drop(columns=wiki_column, inplace=True)
~~~~

After this was finished, all that was left was to rename certain columns and load the dataset into PostgreSQL.

The ratings csv was much faster to work with since it only needed to be grouped by movieId and then merge with the kaggle information on the Kaggle_ID columns.

## Results

![Database_Loading](/Resources/Runtime.png)

After the Extraction and Transformation were finished, all that was left was to Load the data into a table using the sqlalchemy dependency as shown in the image.