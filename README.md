# Udacity Project: Data Wrangling Project

## Nanodegree: Data Analyst

## Title: Wrangle and Analyze Data

### Table of Contents

<li><a href="#project_motivation">Project Motivation</a></li>
<li><a href="#file_descriptions">File Descriptions</a></li>
    <ul>
    <li><a href="#data_files">Data Files</a></li>
    <li><a href="#doc_files">Doc Files</a></li>
    <li><a href="#data_files">Tools</a></li>
    </ul>
<li><a href="#working_proccess">Working Proccess</a></li>
    <ul>
    <li><a href="#gather">Gathering The Data</a></li>
    <li><a href="#assess">Assessing The Data</a></li>
        <ul>
        <li><a href="#quality">Quality</a></li>
        <li><a href="#tidiness">Tidiness</a></li>
        </ul>
    <li><a href="#cleaning">Cleaning The Data</a></li>
    <li><a href="#saving">Saving The Results In A Database</a></li>
        <ul>
        <li><a href="#final_df">Final dataframes afther the cleaning part</a></li>
        <li><a href="#tables">Tables Stored In The Database</a></li>
        </ul>
    <li><a href="#analysis">Analysis</a></li>
        <ul>
        <li><a href="#analyzing_conversations">Analysis Of Conversations In The Replies</a></li>
        <li><a href="#analyzing_proportions">Analysis Of The Proportions Of Dogs Detected In The Pictures</a></li>
        <li><a href="#analyzing_replies">Analysis Of Replies Per User And Per Breed</a></li>
        </ul>
    <li><a href="#future">Future Analysis</a></li>
    </ul>


<a id='project_motivation'></a>
## Project Motivation

The aim of this project is to gatter data from different sources related with the Twitter account `@dog_rates`, to wrangled them and to make a basic analysis.

We have a variety of different sources to extract the data: csv and tsv files provided by Udacity, the tweeter API `tweepy`, and other scrapping methods like using the library `urllib3` to download and to scrap the html code or using librarys as `selenium` to use the local web broweser to download and to read the pages.   



<a id='file_descriptions'></a>
## File Descriptions


<a id='data_files'></a>
### Data Files:

* **resources/twitter-archive-enhanced.csv:**  A csv file with 2356 tweets of this account. 

* **resources/image-predictions.tsv:** A tsv file with the results obtained of applying a predictive method over the pictures of the tweets. 

* **resources/twitter_archive.json:** file constructed via API with the html code of all the tweets contained in `twitter-archive-enhanced.csv` that are still available.

* **resources/twitter_archive_indent.json:** the same file as before, but idented to make easier the visual analysis.

* **resources/scrapped_replies.csv:** replies to each tweet scrapped using selenium.

* **resources/twitter_archive_master.csv:** combined and cleaned data with the information of `twitter-archive-enhanced.csv`, the information extracted with tweepy and the predictions file.  

* **resources/we_rate_dogs.db:** A database in sqlite with the cleaned information of the sources before. Each final dataframe is saved into a table.

<a id='doc_files'></a>
### Doc Files:

* **wrangle_act.html and wrangle_act.ipynb:** code for gathering, assessing, cleaning, analyzing, and visualizing data.

* **wrangle_report.html and wrangle_report.ipynb:** documentation for data wrangling steps: gather, assess, and clean.

* **act_report.html and act_report.ipynb:** documentation of analysis and insights into the final data.

<a id='tools'></a>
### Tools:

* **geckodriver.exe:** a driver to do scrapping using selenium.



<a id='working_proccess'></a>
## Working Proccess


<a id='gather'></a>
### Gathering The Data

We have worked with four data sources:

* **'twitter-archive-enhanced.csv'**: A csv file with 2356 tweets of this account. Each one with a picture of a dog. This file is provided by Udacity for making the project. We have read the file and we have stored it in ha dataframe called df_twitter_archive_enhanced.

* **'image-predictions.tsv':** A tsv file with the results obtained of applying a predictive method over the pictures of the tweets. This file was obtained in a project in another nanodegree and it is provided by Udacity also. Every image in the WeRateDogs Twitter archive was run through a neural network that can classify breeds of dogs. The results: a table full of image predictions (the top three only) alongside each tweet ID, image URL, and the image number that corresponded to the most confident prediction (numbered 1 to 4 since tweets can have up to four images). Udacity gave as the url where we can find this file, so se have download the file to the local PC and we have store the data into another dataframe called df_image_predictions.

* **Additional information obtained with tweepy:** Once obtained the Twitter credentials, we have used the tweepy API to get more additional data. To do this we have connected to the Twitter platform and using tweepy we have downloaded the tweet status for each tweet in twitter-archive-enhanced.csv. Then we have saved these results in a file called twitter_archive.json using the json library. Finally, we have read this file and we have extracted some more data to another dataframe called df_tweepy_extractions using json again.

* **Information about the replies of each tweet:** Finally we have extracted the data corresponding to the replies for each tweet in twitter-archive-enhanced.csv. We have tried some different methods like the tweepy API itself, the library urllib3 or selenium. Finally, I decided to use the last option because I found easier to do scrolling and get all the replies to a tweet.  When there are too many replies, the page cut the list and ask you in a link at the end if you want to see more. You had to do this as many times as you need until you reach the end of the list. I wasn't very sure how to do this with urllib3 and tha API has its limitations also.

<a id='assess'></a>
### Assessing The Data

In this step we have assessed each dataframe with objective of finding possible issues to clean. We have grouped this issues into **Quality** issues and **Tidiness** issues:

<a id='quality'></a>
#### Quality

##### `df_twitter_archive_enhanced` table:

- Some tweets are not available now. 
- The type of the columns `tweet_id`, `in_reply_to_status_id`, `in_reply_to_user_id`, `retweeted_status_id`, `retweeted_status_user_id` should be string.
- We are not intereste in the first par of the text (RT @XXXX:) when the tweet is a retweet. I already have this information in other columns.
- There are 23 rows with rating_denominator different to 10.
- And many rows have a numerator not very realistic.
- The name of the dogs `a`, `O`, `by`, `an`, `the`, `his`, `all`and `my` are incorrect. (We are not going to correct this by the moment. We do not need them)
- There are not many dogs classified as doggo, floofer, etc. And 14 of them have double clasification.
- We would like to know the number of replies to each tweet.


##### `df_image_predictions` table:

- The type of the column `tweet_id` should be string.


##### `df_tweepy_extractions` table:

- The column `retweet_count_retweet` has no sense because it has the same value as `retweet_count`.
- we don't need the columns `entities_name`, `entities_screen_name`, `entities_type`, `mentions_name` and `mentions_screen_name` because we are going to use only the ids, so we can drop them by now.
- Nulls represented as void strings in `entities_name`,	`entities_screen_name`, `entities_type`, `entities_user_id`,	`entions_name`,	`mentions_screen_name`,	`mentions_user_id`,	`quoted_status_id`,	`quoted_status_id_rwetweet` and `retweet_count_retweet`.


##### `df_scrapped_replies` table:

- we don't need the columns `user_name` and `full_name` because we are goin to user only the id.
- The type of the columns `favs`, `replies` and `retweets` should be integer instead of float.
- The type of the columns `user_id` and `reply_id` should be a string.
- `language` type should be categorical.


<a id='tidiness'></a>
#### Tidiness

##### `df_twitter_archive_enhanced` table:

- Columns `doggo`, `floofer`, `pupper` and `puppo` should be a unique column called dog_type.


##### `df_image_predictions` table:

- this dataframe should be integrated in df_twitter_archive_enhanced.

##### `df_tweepy_extractions` table:

- this dataframe should be integrated in df_twitter_archive_enhanced.

##### `df_scrapped_replies` table:


##### `df_twitter_archive_enhanced` and `df_scrapped_replies` 

- We want mentions, replies, retweets, quoted, etc grouped apart as a different kind of information. We can drop this columns in the original dataframes. We will do this at the end of the clean work.

<a id='cleaning'></a>
### Cleaning The Data

Next we have clean the issues mentioned before trying to follow the order: Missing Data, Tidiness and Quality.

We have also followed the same proccess to solve each issue. 

  * Firstly, we have **defined** the issue.
  
  * Then we have **code** the solution.
  
  * Finally, we have **test** te data cleaned to confirm that the issue was solved.


<a id='saving'></a>
### Saving The Results In A Database

* Finally we have saved the final dataframes in a local file called `we_rate_dogs.db` using the library sqlite3 for a further analysis.

<a id='final_df'></a>
#### Final dataframes afther the cleaning part:

**df_twitter_archive_enhanced_copy**

* **tweet_id (String):** The integer representation of the unique identifier for this Tweet. 
* **timestamp (String):** date and time of the tweet.
* **source (String):** Utility used to post the Tweet, as an HTML-formatted string. Tweets from the Twitter website have a source value of web.
* **text (String):** The actual UTF-8 text of the status update. 
* **expanded_urls (String):** url of the tweet.
* **rating_numerator (Float64):** numerator of the rating assigned according to the text of the tweet.
* **rating_denominator (Float64):** denominator of the rating assigned according to the text of the tweet.
* **name (String):** name of the dog according to the text of the tweet.
* **dog_type (String):** type of the dog acording to the text and to the clasification used in the page:'doggo', 'fluffer', 'pupper' and 'puppo'.
* **replies_count (Int64):** Number of replies to each tweet in @dog_rates account.
* **retweet_count (Float64):** Number of times this Tweet has been retweeted.
* **favorite_count (Float64):** Indicates approximately how many times this Tweet has been liked by Twitter users.
* **favorites_count_retweet (Float64):** This field only surfaces when the Tweet is a retweet. Indicates approximately how many times the original Tweet has been liked by Twitter users. 
* **jpg_url (String):** The url of the image of the tweet. It can be downloaded. 
* **img_num (Int64):** The image with the most confident prediction.
* **p1 (String):** I is the algorithm's #1 prediction for the image in the tweet.
* **p1_conf (Float64):** It is how confident the algorithm is in its #1 prediction.
* **p1_dog (Int64):** It is whether or not the #1 prediction is a breed of dog.
* **p2 (String):** It is the algorithm's second most likely prediction.
* **p2_conf (Float64):** It is how confident the algorithm is in its #2 prediction.
* **p2_dog (Int64):** It is whether or not the #2 prediction is a breed of dog.
* **p3 (String): It is the algorithm's third most likely prediction.
* **p3_conf (Float64):** It is how confident the algorithm is in its #3 prediction.
* **p3_dog (Int64):** It is whether or not the #3 prediction is a breed of dog.

**df_scrapped_replies_copy**

* **conversation (String):** Id of the replied tweet.
* **favs (Int64):** Number of favorites for this replying tweet.
* **image (String):** If there is an image in the reply, it especifies the url.
* **language (String):** When present, indicates a BCP 47 language identifier corresponding to the machine-detected language of the Tweet text.
* **references (String):** Other users ID that are referenced in the text of the reply, if they exist.
* **replies (Int64):** Number of replies to this reply.
* **reply_id (String):** tweet ID for this reply.
* **retweets (Int64):** Number of retweets of this reply.
* **text (String):** Text include in the reply.
* **timestamp (String):** Date_time of the reply.
* **user_id (String):** Id of the user who has replied.


**df_interactions**

* **tweet_id (String):** Id of the tweet where the interaction between users is located.
* **user_destiny (String):** User ID who is subject of the interaction: The user mentioned, the user who has been replied, the user whose tweet has been quoted, etc.
* **tweet_id_origin (String):** Tweet ID of the tweet that has been replied, quoted, etc. If the reply is a reply to another reply, we store the ID of the first original tweet. In this case the ID of the tweet from @rate_dogs that originated the conversation.
* **user_origin (String):** User ID who is object of the interaction: The user who mentions, the user who replies, the user whose tweet is a quote, etc.
* **interaction_type (String):** Different types of interactions between users: 'in_reply_to_user_id', 'retweeted_status_user_id', 'quoted_user_id', 'entities_user_id', 'mentions_user_id'and 'references_to_user'.



<a id='tables'></a>
#### Tables Stored In The Database:

This are the name of the tables stores int de DBLite database with the content indicated in the previous dataframes.

* **twitter_master**

* **scrapped_replies**

* **interactions**

<a id='analysis'></a>
### Analysis

In this case the analysis is quite simple, since the main objective of the project was to be able of gatter and clean data from multiple data sources following a structured procedure.

Anyway, this was the different analysis that we have done:

<a id='analyzing_conversations'></a>
#### Analysis Of Conversations In The Replies

We have tried to find long conversations between users. This coud be useful for some kind of applications.
We have extracted some of this conversations using the mentions that the users made to other users in their replies. Especifically we have filtered those replies in a conversation where one single user have mentioned another user more than five times in the same conversation. We mean as 'conversation' to all the replies to a post or tweet of the principal page @dog_rates.

| Original User      | User Mentioned     | Conversation       | Mentions |
|--------------------|--------------------|--------------------|----------|
| 80630279           | 716594909709512705 | 874296783580663808 | 12       |
| 160774025          | 111835051          | 672609152938721280 | 10       |
| 274083242          | 22750218           | 817827839487737858 | 10       |
| 274083242          | 22750218           | 836397794269200385 | 10       |
| 3303277983         | 4519686269         | 830097400375152640 | 7        |
| 3303277983         | 865851604838354945 | 830097400375152640 | 7        |
| 716594909709512704 | 80630279           | 874296783580663808 | 7        |
| 1130081593         | 47055321           | 878604707211726852 | 6        |
| 22750218           | 274083242          | 817827839487737858 | 6        |
| 22750218           | 274083242          | 836397794269200385 | 6        |
| 2801001643         | 538278235          | 814578408554463233 | 6        |


<a id='analyzing_proportions'></a>
#### Analysis Of The Proportions Of Dogs Detected In The Pictures


In this part we have investigated how good or bad are the prediction methods. Expecifically, we have studied if they are good predicting whether the picture is of a dog or not. To know this we have extracte a random sample of 300 pictures and check them by ourserlves. We have decided if there is a dog in the picture or not and we have saved it in the dataframe as a boolean value. Our criteria is very subjective, we have decided to put a yes in all cases that there is a dog in the picture, even if it is not the main objective of the picture, if there are other animals in front of them or if they are too far or blurred. But it can be a reference to compare with the methods provided in the file `image-predictions.tsv`.

Once that we have this 300 pictures we have done bootstrapping to calculate the range of proportion of dogs in all the pictures with a confidence interval of 95%, and compare this values with the proportions given by the predictors:

<img src=pictures/dist_prop.jpg alt="drawing" width="700"/>

The values obtained for this sample have been:

* **Proportion of dogs observed:** 0.93.
* **Margin of proportions for the population with a confidence interval of 95%:** 0.9 to 0.957. 


Wile the values for the proportion of dogs detected with the three methods given by Udacity are:

* **Method 1:** 0.735.
* **Method 2:** 0.75.
* **Method 3:** 0.744.

If we consider the case in which any of the three methos have detected that the picture is about a dog:

* **Method 1 OR Method 2 OR Method 3:** 0.844. 

We have seen that in all the cases they are too far from the confidence interval that we have obtained before. 


we have also applied a logistic regression method to know with predictive method can be the better to guess whether there is a dog in the picture or not. In this case our observed value in `real_dog` column will be the response variable and the predictors will be `intercept`,`p1_dog`, `p2_dog` and `p3_dog`.

This has been the result:


![](pictures/Logit_regression_results.png)


Doing the exponential of the coefficients to see their influence in the linear regression of the logit:

intercept: 1.71821559

p1_dog: 7.50934685

p2_dog: 2.4116294

p3_dog: 4.26207906


We have concluded from this values that teh P1 is the method that is more closer to our interpretations, because if this method detects a dog, the logit of the probability of that we have detected a dog is incremented by 7.5. While with the method P2 it only increments by 2.4 and by 4.3 in the case of P3.

<a id='analyzing_replies'></a>
#### Analysis Of Replies Per User And Per Breed

We have explore the distribution of the number of replies per user and we have saw that most people only comment one or two times on the page. The best way to show that is with the description of this distribution, because the graphic is too highly skewed to the right. 

|     .  | **Replies**  |
|----------:|----------|
| **Count** | 22100    |
| **Mean**  | 1.769683 |
| **Std**   | 3.265758 |
| **Min**   | 1        |
| **25%**   | 1        |
| **50%**   | 1        |
| **75%**   | 2        |
| **Max**   | 148      |

To make this table we have delete all the replies corresponding to the user @dog_rates, that obviusly has the highest number of replies (465), but we are more interested in the behaviour of the rest of the users.

We can see that the 75% of all the users only have written one or two replies. And the standar deviation is only 3.

Anyway there are some users that have been very active in this account. For example if we extract the ten users with more comments in the tweets, we will have this table:

|    User ID | Replies |
|-----------:|---------|
| 4196983835 | 465     |
| 401334573  | 148     |
| 21163853   | 134     |
| 594710756  | 108     |
| 265825342  | 101     |
| 27603608   | 92      |
| 1432492135 | 85      |
| 1337271    | 79      |
| 842618414  | 76      |
| 1417162219 | 67      |

Here we can see, as we said, that the user @dog_rates is who have commented more pictures. And the next one has written 148 replies in all the registered tweets.

<a id='future'></a>
### Future Analysis

We could implement some kind of NLP method to detect if the pictures of a tweet make reference to a dog or if it is some kind of joke. For example, a man disguised, a dog disguised of a seal, etc. We have a lot of replyes and they can be used to differentiate real comments and ratings of dogs from those tweets where people only wanted to be funny.


