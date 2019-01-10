# Reddit Recommender
## Introduction
The goal of this project is to find similar subreddits to a user-provided subreddit. Specifically, the focus is on posts with images, in order to eventually find images with similar content.

## Data & Preprocessing
This project uses https://github.com/praw-dev to interact with reddit. 
Retrieving comments does not work yet, due to a memory leak. Images are downloaded via https://pypi.org/project/wget/.
All preprocessing can be found in `preprocess_data.ipynb`. 

## Model Discussion
The problem can be phrased and viewed in at least the following ways.

As a recommender system. In this setting items are recommended to users. Typically, one has information about interaction between users and items, eg. we could collect data on users following subreddits and then based on the subreddits they already follow predict which new subreddits are of their interest. With the data given here (only posts), one could only do content based recommendation. Overall this seems slightly non-ideal for the given data. 

Multi-class classification. In a classification setting the goal is to assign new data the most likely category, given observed/labeled data. Here this would mean to assign either each post title, or the collection of all post titles their subreddits as category. During prediction one would either use a new post title (eg. a user input) to classify, or the text of a whole subreddit. The output(s) would be the closest matches in terms of subreddits. 

Text similarity
