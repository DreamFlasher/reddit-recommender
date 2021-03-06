# Reddit Recommender
## Introduction
The goal of this project is to find similar subreddits to a user-provided subreddit. Specifically, the focus is on posts with images, in order to eventually find images with similar content.

## Data & Preprocessing
This project uses https://github.com/praw-dev to interact with reddit. 
Retrieving comments does not work yet, due to a memory leak. Images are downloaded via https://pypi.org/project/wget/.
All preprocessing can be found in `preprocess_data.ipynb`. 

## Model Discussion
The problem can be phrased and viewed in at least the following ways.

(1) As a recommender system. In this setting items are recommended to users. Typically, one has information about interaction between users and items, eg. we could collect data on users following subreddits and then based on the subreddits they already follow predict which new subreddits are of their interest. With the data given here (only posts), one could only do content based recommendation. Overall this seems slightly non-ideal for the given data. 

(2) Text similarity. In this category I would put all methods that calculate a similarity or distance score for two texts. The most common ones are Jaccard Similarity (counting overlapping words), Cosine Similarity with TF-IDF or word2vec. This looks like an idea fit for the problem at hand; get a vector representation for each subreddit or post, and then get the most similar subreddits according to the chosen metric. 

(3) Multi-class classification. In a classification setting the goal is to assign new data the most likely category, given observed/labeled data. Here this would mean to assign either each post title, or the collection of all post titles their subreddits as category. During prediction one would either use a new post title (eg. a user input) to classify, or the text of a whole subreddit. The output(s) would be the closest matches in terms of subreddits.

## Model Implementation & Evaluation
I decided to implement one model of each (2) and (3). 

For (2) I chose the doc2vec model of gensim (https://github.com/RaRe-Technologies/gensim/blob/develop/gensim/models/doc2vec.py). Le and Mikolov (["Distributed Representations of Sentences and Documents"](http://arxiv.org/pdf/1405.4053v2.pdf)) demonstrated that their technique to calculate document vectors generally outperforms calculating a document vector by summing all word vectors. 

More details, including the possibility to actually query the models can be found here `recommend_subreddits.ipynb`. The evaluation shows that when querying subreddits with a sufficient number of posts (>100 posts in the evaluation data), this method provides the expected results. In order to use it with user-specified query strings, further improvements are necessary, aso shown in the same notebook. The models are trained in about 2 hours, and the prediction is sufficiently fast. 

For (3) I picked fastText (https://github.com/facebookresearch/fastText). In the end the models I chose for (2) and (3) are very similar, and it would be a reasonable next step to compare the performance to models based on tf-idf. 

This model is inherently suitable for string-based queries, as it classifies each sentence (post title). Use the following command to train on the fasttext formatted files from the preprocessing notebook:

`./fasttext supervised -input imgposts_fasttext.train -output model_imgposts_fasttext -epoch 25 -wordNgrams 4 -loss hs` 

The `hs` loss is the hierarchical sampling loss, which speeds up training considerably when there are many classes. And the following for evaluation:

`./fasttext test model_imgposts_fasttext.bin imgposts_fasttext.valid`

This achieves a precision and recall of about 30%. In order to query the model with a string, create a text file with the query and then execute the following command:

`echo "sonic screwdriver" >> query.txt`

`./fasttext predict-prob model_imgposts_fasttext.bin query.txt 5`

results in a very reasonable: `__label__doctorwho 0.79995 __label__mattsmith 0.181585 __label__DoctorWhumour 0.00714386 __label__BowlofLemons 0.0035089 __label__Staples 0.00225036`

## Future Work
I demonstrated that the model based on doc2vec is suitable for finding similar subreddits, given a subreddit. If a user desires to query by keywords, the fasttext model is more suitable. In future there should be one model, for the different query possibilities. 

I retrieved all images for the posts (see the preprocessing notebook), and I wanted to run a hierarchical image clustering algorithm based on vgg, to investigate overlaps between subreddits and clusters. This would allow to verify if subreddits are a sufficient indicator of image content. 

I also retrieved most comments of the posts, but did not continue further investigation into this, as there was a memory leak in the library to retrieve the comments. The extension of the both models to include this data is straightforward.

Negative subreddits or queries can easily be achieved with doc2vec too, by calculating the vector represenation of the negative/excluding subreddit/query and then subtracting this from the positive subreddit/query and then calculating the most similar vectors (tags) to this results vector. 

In order to answer if the eventual goal of locating similar images is achievable via text similarity, eg. if text is a good proxy for image similarity, I would like to evaluate the image clustering first. If the clusters are randomly distributed over the subreddits, then not. I remain doubtful that this will work well. Overall there are too many open questions to me: What kind of similarity is desired? How is the similarity measured? I expect a lot of noise in the data, both in the texts and the pictures. Why not use existing object detection algorithms on the images itself?
