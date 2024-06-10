# Movie Database Management

This project involves managing a movie database using SQLite. It includes creating tables, inserting data, and performing various queries to extract interesting information from the dataset. The dataset used is the IMDB 5000 Movie Dataset.

## Table of Contents

- [Introduction](#introduction)
- [Dataset](#dataset)
- [Database Schema](#database-schema)
- [Setup Instructions](#setup-instructions)
- [Usage](#usage)
- [Queries and Operations](#queries-and-operations)
- [License](#license)

## Introduction

The purpose of this project is to transition from an Excel-based dataset to an SQL-based database, using SQLite. This will help in efficiently managing and querying movie data. The project demonstrates the creation of database tables, insertion of data, and execution of various SQL queries.

## Dataset

The dataset used is the [IMDB 5000 Movie Dataset](https://data.world/data-society/imdb-5000-movie-dataset). It contains information about movies such as their titles, directors, duration, genres, and various ratings.

## Database Schema

### Tables

1. **Movie**
    - `movie_id`: INTEGER PRIMARY KEY AUTOINCREMENT
    - `movie_title`: TEXT
    - `director_name`: TEXT
    - `duration`: REAL
    - `genres`: TEXT
    - `content_rating`: TEXT
    - `budget`: REAL
    - `title_year`: REAL

2. **IncomeAndSpend**
    - `movie_id`: INTEGER
    - `gross`: REAL
    - `budget`: REAL
    - `FOREIGN KEY(movie_id) REFERENCES Movie(movie_id)`

3. **Ratings**
    - `movie_id`: INTEGER
    - `imdb_score`: REAL
    - `num_critic_for_reviews`: REAL
    - `num_user_for_reviews`: REAL
    - `movie_facebook_likes`: REAL
    - `FOREIGN KEY(movie_id) REFERENCES Movie(movie_id)`

## Setup Instructions

### Prerequisites

- Python 3.x
- SQLite 3.x
- Pandas library

### Steps

1. Clone the repository:
    ```bash
    git clone https://github.com/yourusername/movie-database-management.git
    cd movie-database-management
    ```

2. Install required Python packages:
    ```bash
    pip install pandas sqlite3
    ```

3. Download the dataset and place it in the project directory:
    - [IMDB 5000 Movie Dataset](https://data.world/data-society/imdb-5000-movie-dataset)

4. Run the setup script to create the database and insert data:
    ```bash
    python setup_database.py
    ```

## Usage

### Running Queries

You can execute SQL queries directly using the SQLite command-line tool or through a Python script. Below is an example Python script to run a query:

```python
import sqlite3

# Connect to the SQLite database
conn = sqlite3.connect('movies.db')
cursor = conn.cursor()

# Example query: List all movies with IMDB score greater than 7
query = '''
SELECT movie_title 
FROM Movie 
JOIN Ratings ON Movie.movie_id = Ratings.movie_id 
WHERE imdb_score > 7;
'''

cursor.execute(query)
results = cursor.fetchall()

for row in results:
    print(row)

# Close the connection
conn.close()
Viewing Tables
To list all tables in the database:

sql
Copy code
SELECT name FROM sqlite_master WHERE type='table';
To describe a specific table:

sql
Copy code
PRAGMA table_info(Movie);
Queries and Operations
Select Movies with Actor Names, Director, and Gross Revenue
sql
Copy code
SELECT M.movie_title, M.director_name, I.gross
FROM Movie M
JOIN IncomeAndSpend I ON M.movie_id = I.movie_id
ORDER BY I.gross DESC;
Select Top 3 Highest-Grossing Movies
sql
Copy code
SELECT M.movie_title, I.gross
FROM Movie M
JOIN IncomeAndSpend I ON M.movie_id = I.movie_id
ORDER BY I.gross DESC
LIMIT 3;
Movies with IMDB Score Between 6 and 7
sql
Copy code
SELECT M.movie_title
FROM Movie M
JOIN Ratings R ON M.movie_id = R.movie_id
WHERE R.imdb_score BETWEEN 6 AND 7;
Update Facebook Likes for a Specific Movie
sql
Copy code
UPDATE Ratings
SET movie_facebook_likes = 7500
WHERE movie_id = 5;
Transaction: Update and Rollback Facebook Likes
sql
Copy code
BEGIN TRANSACTION;
UPDATE Ratings
SET movie_facebook_likes = 4700
WHERE movie_id = 7;
ROLLBACK;
Create View Combining Rating Data with Movie Data
sql
Copy code
CREATE VIEW MovieRatings AS
SELECT M.movie_title, M.director_name, R.imdb_score, R.num_critic_for_reviews, R.num_user_for_reviews, R.movie_facebook_likes
FROM Movie M
JOIN Ratings R ON M.movie_id = R.movie_id;
Average Budget and Average Income of All Movies
sql
Copy code
SELECT AVG(I.budget) AS avg_budget, AVG(I.gross) AS avg_income
FROM IncomeAndSpend I;
Movies with IMDB Ratings More Than 7 and Facebook Likes Less Than 20000
sql
Copy code
SELECT M.movie_title
FROM Movie M
JOIN Ratings R ON M.movie_id = R.movie_id
WHERE R.imdb_score > 7 AND R.movie_facebook_likes < 20000;
License
This project is licensed under the MIT License. See the LICENSE file for details.

python
Copy code

### Additional File: `setup_database.py`

Include this script in your project directory to set up the SQLite database with the movie data:

```python
import pandas as pd
import sqlite3

# Load the CSV file
file_path = 'movie_metadata.csv'
movie_data = pd.read_csv(file_path)

# Establish SQLite connection
conn = sqlite3.connect('movies.db')
cursor = conn.cursor()

# Create tables
cursor.execute('''
CREATE TABLE IF NOT EXISTS Movie (
    movie_id INTEGER PRIMARY KEY AUTOINCREMENT,
    movie_title TEXT,
    director_name TEXT,
    duration REAL,
    genres TEXT,
    content_rating TEXT,
    budget REAL,
    title_year REAL
)
''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS IncomeAndSpend (
    movie_id INTEGER,
    gross REAL,
    budget REAL,
    FOREIGN KEY(movie_id) REFERENCES Movie(movie_id)
)
''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS Ratings (
    movie_id INTEGER,
    imdb_score REAL,
    num_critic_for_reviews REAL,
    num_user_for_reviews REAL,
    movie_facebook_likes REAL,
    FOREIGN KEY(movie_id) REFERENCES Movie(movie_id)
)
''')

# Insert data into tables
for index, row in movie_data.iterrows():
    cursor.execute('''
    INSERT INTO Movie (movie_title, director_name, duration, genres, content_rating, budget, title_year)
    VALUES (?, ?, ?, ?, ?, ?, ?)
    ''', (row['movie_title'], row['director_name'], row['duration'], row['genres'], row['content_rating'], row['budget'], row['title_year']))
    
    movie_id = cursor.lastrowid
    
    cursor.execute('''
    INSERT INTO IncomeAndSpend (movie_id, gross, budget)
    VALUES (?, ?, ?)
    ''', (movie_id, row['gross'], row['budget']))
    
    cursor.execute('''
    INSERT INTO Ratings (movie_id, imdb_score, num_critic_for_reviews, num_user_for_reviews, movie_facebook_likes)
    VALUES (?, ?, ?, ?, ?)
    ''', (movie_id, row['imdb_score'], row['num_critic_for_reviews'], row['num_user_for_reviews'], row['movie_facebook_likes']))

# Commit changes and close connection
conn.commit()
conn.close()
