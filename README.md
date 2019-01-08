# Reddit Recommender
## Introduction
The goal of this project is to find similar subreddits to a user-provided subreddit. Specifically, the focus is on posts with images, in order to eventually find images with similar content.

## Data & Preprocessing
This project uses https://github.com/praw-dev to interact with reddit. 
Retrieving comments does not work yet, due to a memory leak. Images are downloaded via https://pypi.org/project/wget/.
All preprocessing can be found in `preprocess_data.ipynb`. 

## Model Discussion
The problem can be phrased and viewed in at least the following ways.

### Recommender
In a recommender system 

### Multi-class classification

### Text similarity
