# Book Recommendation System

A book recommender built in a single Jupyter notebook. It combines two approaches:

1. **Popularity-based recommender**: surfaces the top 50 books by average rating among books that have been rated many times.
2. **Collaborative filtering recommender**: given a book title, returns the 5 most similar books using item-based cosine similarity over user ratings.

## Project Structure

```
.
├── BookRecommendationSystem.ipynb   # Full analysis and both recommenders
├── dataset/
│   ├── Books.csv                    # Book metadata
│   ├── Users.csv                    # User location and age
│   └── Ratings.csv                  # User-book ratings
└── README.md
```

## Dataset

The notebook expects three CSV files inside a `dataset/` folder (Book-Crossing format):

| File | Rows | Columns |
|------|------|---------|
| `Books.csv` | 271,360 | ISBN, Book-Title, Book-Author, Year-Of-Publication, Publisher, Image-URL-S, Image-URL-M, Image-URL-L |
| `Users.csv` | 278,858 | User-ID, Location, Age |
| `Ratings.csv` | 1,149,780 | User-ID, ISBN, Book-Rating |

**Data quality checks**

- No duplicate rows in any of the three files.
- `Books.csv`: 2 missing `Book-Author`, 2 missing `Publisher`, 3 missing `Image-URL-L`.
- `Users.csv`: 110,762 missing `Age` values (about 39.7%).
- `Ratings.csv`: no missing values.

## Approach

### 1. Popularity-Based Recommender

1. Merge `Ratings` with `Books` on `ISBN` (1,031,136 rated rows remain after the join).
2. Compute the number of ratings and the average rating per `Book-Title`.
3. Keep only books with **at least 250 ratings**, then sort by average rating and take the **top 50**.
4. Join back to `Books` for author, year and cover image, and drop duplicate editions (the same title can appear under several ISBNs).

The final table has: `Book-Title`, `Book-Author`, `Year-Of-Publication`, `Image-URL-M`, `No_of_rating`, `Average_rating`.

### 2. Collaborative Filtering Recommender

1. **Active users**: keep users who rated more than 200 books (811 users).
2. **Well-rated books**: among those users' ratings, keep books with at least 50 ratings (706 books, 58,586 ratings).
3. **Pivot table**: build a book x user matrix of ratings (`Book-Title` as rows, `User-ID` as columns) and fill missing values with 0.
4. **Similarity**: compute pairwise cosine similarity between books using `sklearn.metrics.pairwise.cosine_similarity`.
5. **Recommend**: look up the book's row in the similarity matrix and return the 5 most similar titles (excluding the book itself).

## Usage

Open the notebook and run all cells, then call:

```python
recommend('Harry Potter and the Chamber of Secrets (Book 2)')
```

Output:

```
Harry Potter and the Prisoner of Azkaban (Book 3)
Harry Potter and the Goblet of Fire (Book 4)
Harry Potter and the Sorcerer's Stone (Harry Potter (Paperback))
Harry Potter and the Sorcerer's Stone (Book 1)
Harry Potter and the Order of the Phoenix (Book 5)
```

The title must exactly match an entry in the pivot table's index (one of the 706 books that passed the filters).

## Getting Started

**Requirements**

- Python 3.11 (the notebook was run on 3.11)
- pandas
- numpy
- scikit-learn
- Jupyter

**Setup**

```bash
# 1. Clone the repository and move into it
git clone https://github.com/ArunAryal/Book-Recommendation-System
cd Book-Recommendation-System

# 2. Install dependencies
pip install pandas numpy scikit-learn jupyter

# 3. Put Books.csv, Users.csv and Ratings.csv inside a dataset/ folder

# 4. Launch the notebook
jupyter notebook BookRecommendationSystem.ipynb
```

## Notes and Limitations

- In this dataset a rating of `0` usually means the user interacted with the book without giving an explicit score. Filling missing values with `0` in the pivot table treats "not rated" and "rated 0" the same way.
- Popularity ranking uses the mean of all ratings, including `0`s, so averages are lower than the usual 1-10 scale would suggest.
- The recommender only covers the 706 books that pass the filtering thresholds; other titles raise an error in `recommend()`.
- Similarity is item-based on titles only, so different editions of the same book are merged in the collaborative filtering step.
