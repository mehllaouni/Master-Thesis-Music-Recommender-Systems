[Uploading El-Mehdi-Ellaouni-Master-Thesis-Recommender-Systems-for-music.pdf…]()
# Master-Thesis-Music-Recommender-Systems

```
September 2020
```
## Abstract

To navigate the ever-expanding pool of music libraries, recommender systems (RS) have become a
necessary tool for online music platforms, and sometimes even a competitive advantage. This paper
is an overview of music recommendation systems (MRS) and explores the topic from different
angles.

Three types of approaches for MRS have been identified: collaborative filtering (CF), content-based
filtering (CBF), and hybrid. Before comparing the approaches through literature review and
experimental analysis, we first look at the business implications of RS, and the domain specific
challenges they face for music. We present the example of Spotify’s recommendation algorithm as a
successful implementation, then construct our own hybrid system to recommend Spotify music,
using a mix of social filtering, k Nearest Neighbours classifier CBF model, and cosine similarity-based
CF. Finally, beyond the technical limitations, we explore some of the ethical issues raised by MRS.


## CONTENTS


- Abstract
- Introduction........................................................................................................................................
- 1 Chapter
- Importance of MRS for Music Streaming Companies
   - 1 .1 Specific challenges of music recommendation algorithms
      - 1 .1.1 Overview of Music Recommendation Systems (MRS)
      - 1 .1.2 Music dependant challenges
      - 1 .1.3 Interface dependant challenges
   - 1 .2 Business stakes of recommendation systems
- 2 Chapter
   - 2 .1 Overview of the MRS techniques
      - 2 .1.1 Content-Based Filtering
      - 2 .1.2 Collaborative Filtering
      - 2 .1.3 Hybrid Recommenders...............................................................................................
      - 2 .1.4 Evaluation of music recommendation systems
   - 2 .2 A case study of Spotify
- 3 Chapter
- Building a hybrid recommender system for spotify music
   - 3 .1 Framework
   - 3 .2 Data collection
   - 3 .3 Experimentation
      - 3 .3.1 User-user CF Model
      - 3 .3.2 CBF Model
      - 3 .3.3 Hybrid model
   - 3 .4 Evaluation measures..........................................................................................................
   - 3 .5 Results
      - 3 .5.1 Exploratory phase
      - 3 .5.2 Performance of the Hybrid recommender
   - 3 .6 Step-by-step example
   - 3 .7 Limits of our model and further improvements
      - 3 .7.1 Limits of the existing model
      - 3 .7.2 Additional improvements...........................................................................................
- 4 Chapter
- The ethical impact of recommendation algorithms
   - 4 .1 Deblackboxing the impact of mrs on artists
   - 4 .2 the impact of recommendation algorithms on users
- Conclusion
- References


## Building a hybrid recommender system for spotify music

### 3 .1 Framework

The purpose of this chapter is to build an implementation of a RS to understand what makes the
hybrid method more suited for music recommendation. The idea is to analyse several users’
preferences based on their playlists and build a recommended playlist with **25 new songs** for each
one, using our algorithms.

The hybrid model will be a combination of different approaches explained in the previous chapters.
Three main ideas to build our recommender system:

**1 - Social filter:**

As we have seen in Chapter 1, the social dimension of music is very important, and some RS take the
musical preferences of the user’s **social friends** into account for building better recommendations.
So, for our model, instead of going through a set of random users from the Spotify database, we will
focus on users with close proximity.

We selected 10 users that share the same taste in music. They are all part of the same “clique”, and
members of a Facebook group dedicated to sharing **electronic music** , they listen to music and “dig”
for new songs daily. We extracted all their personal playlists and the playlists they are subscribed to.


For the purpose of this study, this part was done manually but it could eventually be automated
using the ‘Find my friends’ feature on Spotify which allows users to connect to their Facebook
friends’ music accounts.

For each user, we will the use a combination of CF and CBF to recommend songs for each of their
personal playlists.

**2 - Collaborative filtering**

We will use CF to build a similarity matrix between the playlists. Based on each seed playlist, we will
recommend songs from their **most similar playlists**.

**3 - Content based filtering**

After selecting the most similar playlists to recommend from, CBF will determine which songs to
choose for the final recommendation based on item-item similarity by classifying the songs using
their **audio features**.

### 3 .2 Data collection

Spotify allows for developers to connect to their API, to retrieve and manage Spotify data.

After connecting with the proper Spotify credentials and authorizing the collection of data, the data
extraction itself is then done through a Python backend, with the help of the Spotipy library which
supports all the features of the Spotify Web API.

The database is divided in two datasets:

**User playlists** (5415 rows × 22 columns) **—** contains the list of all user playlists, their names, and the
user IDs they belong to


**Playlists tracks** (5 0 rows × 5 columns) **—** contains all the tracks of the playlists, the playlist IDs they
belong to and their audio features. Spotify provides the following attributes for each song:

```
Fig 8. Spotify Audio Features
```
Additional features were also extracted to describe the items.

```
Fig 9. Additional descriptive variables
```

### 3 .3 Experimentation

#### 3 .3.1 User-user CF Model

For this algorithm we will treat the songs as the 'items' and the playlists as the 'users'. The two main
components of this model are:

- **The rating:** Since this metric is not available in Spotify (no way to score a music positively or
negatively and no listening count per music) we will evaluate each song simply by whether it is
included in a playlist.
- **The similarity measure:** The most similar playlists are going to be the ones that include the most
tracks in common.

Our algorithm follows three steps:

1 - Building Playlist-Song sparse matrix as seen in the figure below.

```
Fig 10. Similarity matrix
```

Each row corresponds to a playlist and each column to a song, coded with “1” if the song is in the
playlist and “0” if it is not. The shape of the matrix will be as follows: n (number of playlists) x m
(number of total songs in the playlist)

2 - Computing the similarity between the playlists using the chosen similarity measure. For this
experiment we compare Cosine Similarity and Pearson Correlation.

3 - Selecting the most similar playlists to recommend songs from.

#### 3 .3.2 CBF Model

We will treat this part as a binary classification problem. For each playlist, we will train a classifier to
recognise songs from the seed playlist and then use it to predict if new songs are similar. The two
main components of this model are:

- **The sample data** : Training and testing our classification model on the whole dataset would be
meaningless since we will not have any more data for new prediction. To tackle this, we will use a
subset of the dataset containing the tracks from the seed playlist, corresponding to class “1”, and
twice the number of random songs from the rest of the playlists, corresponding to class “0”. We will
then split this subset into training and testing datasets to tune the classifier.

The sampling is very important as the songs that belong to class “0” will determine how good our
classifier will be. If their features are not different enough from the ones in the seed playlist, our
model will not be able to properly distinguish between the two classes.

- **The classifier** : We will use KNN binary classifier as it is the most common according to the literature
and one of the simplest to implement. We will define a scoring measure to fine-tune our
hyperparameters (number of neighbours and features), but also to evaluate our model’s
performance.


Our algorithm follows these steps:

1 - Sampling the data and creating a working data frame with the classes properly coded.

2 - Splitting the sample into training and testing sets – 80%/20%.

3 - Fine-tuning the number of neighbours using ten-fold cross validation – between 1 and 15
neighbours.

4 - Training a KNN model to classify the songs in the seed playlist.

5 - Predicting the classes on our test set.

6 - Scoring our model using our chosen evaluation measure.

7 - If the scoring measure is satisfying: we will save this model and use it to predict the classes on new
songs from the rest of the data. The positively classified tracks will be the ones with the most similar
audio features as the songs from the seed playlists and will be added to the recommendations list.

8 - If the performance of the model is not satisfying: the algorithm goes back to step one and choses a
new random sample to train and test the model. The whole process will be repeated until we find the
optimal model.

**This process will be repeated for different numbers of features – between 2 and 11 – and finally
the model maximizing the evaluation measure will be chosen.**

#### 3 .3.3 Hybrid model

Our hybrid model will combine both CF and CBF recommenders. It will have the following procedure:

1 - First we will use CF to determine the most similar playlists to the seed playlist.

2 - If no similar playlists are found: CBF is used on the whole dataset to determine songs to
recommend. The hybrid model will then be equivalent to a simple CBF recommender.


3 - If similar playlists are found: instead of using CBF on the whole dataset of new songs, we will only
recommend songs from the most similar playlists determined in the previous steps.

Compared to a single model, it will have two main advantages:

- **Tackling the cold-start problem:** if the CF does not find similar playlists, it means that we do not
have enough information on the playlist to determine similar ones. It either has a few numbers of
tracks or no tracks in common with any other playlist. Using CBF in this case allows us to ignore this
predicament still build a recommendations list.
- **Giving more personalized recommendations** : combing CBF with CF allows for the
recommendations to be more accurate, as they will have the same general features as the ones in
the seed playlist and they will come from similar playlists.

### 3 .4 Evaluation measures..........................................................................................................

The measures we will compare are:

- **CBF Model evaluation** : As we do not have a reliable feedback regarding which items are disliked,
nor numerical ratings to translate user preferences, precision-based metrics, such as RMSE and MSE,
are not very appropriate. Instead, we will use _RECALL_ and _ROC AUC_ to evaluation how good the
model is at distinguishing between classes.

We will run the model twice, each time with optimizing one of the two evaluation measures. The
results of the performance will then be compared, and we will finally choose one of the metrics for
our final model.

We can already suppose that the Recall Maximization Model will be the most satisfying because we
want to have the least number of False Negatives. Having many False Positives is not an issue since
those songs are the ones that are most similar to our base class in the sample data.


- **Similarity measure** : we will use Cosine Similarity, because, according to the literature, it is the most
appropriate when the matrix is too sparse, which is the case for our experiment as the size of the
dataset is not very big.

### 3 .5 Results

In this part we will explore the dataset then present the results of our model and its performance.

#### 3 .5.1 Exploratory phase

The figure below represents the histograms of the audio features distributions.

```
Fig 10. Audio features distributions
```

We observe that while most of the variable are well distributed throughout the playlists, variables
like acousticness, liveness and speechiness are skewed and compact. This is coherent with the fact
that all the tracks in the dataset are from the Electronic Music genre.

**Acousticness** : very skewed to the right and 75 % of songs have an acousticness below 0.071 which
means the majority of the songs are played with electronic rather than acoustic instruments.

**Speechiness** : Third quartile also around 0.071 which means the songs contain little to no lyrics

**Liveness** : Third quartile of 0.89 which means most of the songs were recorded in a studio and not
played live.

We decided to drop those features because they are not going to be very relevant as the classifier
will not be able to differentiate between the tracks regarding them.

To visualize the difference between playlists in terms of average audio features, we select two
playlists _Hard Techno_ and _Old School_ and construct a radar plot with 6 selected features.

```
Fig 11. Radar plot of two playlists
```
We can observe how different from each other the two playlists are. The Old-School playlist tends to
have more popular songs (because they are older and more notorious), with more lyrics and higher
danceability. While the other contains faster songs with more energy.


After the exploration phase, we will compare the results of our different evaluation metrics.

#### 3 .5.2 Performance of the Hybrid recommender

**_3.5.2.1 ROC AUC Maximization Model (AUCM Model)_**
We obtain the following results when choosing to maximize ROC AUC.

**AUC** mean of 0.89

```
Fig 12. Histogram of AUC for AUCM model
```
**Average Precision** mean of 0.8

```
Fig 13. Histogram of Average Precision for AUCM model
```
**Recall score** mean of 0.66


```
Fig 14. Histogram of Recall Score for AUCM model
```
The AUC and Average Precision metrics are satisfactory, but the Recall score is only of 0.66, which
means that on average, when training our classifier, only 66% of positively classified songs are
actually in the seed playlist. Since we want our model to recognize all the playlist’s songs, the recall
score must be further improved.

In addition, the recommender failed to reach the desired objective of 25 recommended songs for 4
out of the 20 playlists. For the playlist ether for example, we were only able to recommend 9 songs.

**_3.5.2.2 Recall Score Maximization Model (RSM Model)_**
We obtain the following results when choosing to maximize the recall score.

**AUC** mean of 0.87

```
Fig 15. Histogram of AUC for RSM model
```

**Average Precision** mean of 0 .74

```
Fig 16. Histogram of Average Precision for RSM model
```
**Recall score** mean of 0.8 1

```
Fig 17. Histogram of Recall Score for RSM model
```
In this model, we get an overall improvement of 0.1 5 without losing too much on the AUC and
precision part (respectively dropped by 0.02 and 0.06).

In addition, the recall maximization model generates more recommended songs due to the False
Positives/False Negatives trade-off, allowing us to reach our goal of 25 recommended songs for each
playlist.

For these reasons, we will choose the _Recall Maximization Model_ as our final recommender.


**3.5.2.3 Exploring the results**
In this section we will explore try to understand the reasons behind the differences in the
performance of the recommender between the playlists. A first line of thought is the size of the
playlist.

**Size of the playlists:**

```
Fig 18. Plots and correlation lines of AUC (red) Precision (green) and Recall (blue) according to the number
of playlists
```
We can observe that there is a negative correlation between the number of tracks of a playlist and
the performance of the classifier. The higher the size of a playlist the less performing the model is,
especially in terms of AUC and Recall Score.


A possible explanation would be that on average, if the number of tracks in a playlist is high, that
means that it tends to contain a broader variety of songs, with different kinds of audio features
which makes them harder to classify.

**Musical coherence:**

We can support the previous findings by introducing the idea of _musical coherence_ within the
playlists:

Having a bigger size of a playlist with a broader variety of songs means the playlist is less musically
coherent, which means the audio features of the playlists are more spread out, and they are also
more likely to have outliers. Therefore, the performance of the classifier drops.

We can further see the impact of musical coherence when analysing the playlists where the
recommender had the highest performances. Let us take the example of the two playlists _Old School_
and _Hard Techno_ , where the classifier scored 1 on all the metrics. As we saw in the radar chart on
FIGUREEE, they are very distinguishable from each other in terms of audio features. In addition, the
names of the playlists are those of two specific sub-genres of electronic music, which means that
their tracks most probably all belong to that same sub-genre, which illustrates their musical
coherence. In comparison, playlists like _ether_ that contain many more songs, and do not seem to
illustrate a specific mood or genre are much harder to classify (AUC of 0.71, Recall Score of 0.42 and
Average Precision of 0.5).

This is confirmed by comparing the boxplots of the playlists with the highest and lowest classification
scores.


```
Fig 19. Boxplots of playlists ether (left) and Old School (right)
```
As we can see the playlist _ether_ has much more outliers and the features are more spread out.

To have a more in-depth vision of our recommender system and its performance, we will analyse
each step through the example of the playlist _Hard Techno_.

### 3 .6 Step-by-step example

**- CF:** After constructing the song/playlist matrix and calculating the cosine similarities, we

obtain 8 playlists that have a positive cosine similarity with our seed playlist, the two most

similar being _Techno Electro_ and _JGS – Techno_ with cosine similarities of respectively 0.210 and

0.102. We will select the most similar for the next step.

- **CBF:** After selecting the sample data and using cross validation to select the best hyperparameters,
the recall maximization model that was selected has 4 neighbours and 6 features – Release date,
Danceability, Energy, Loudness, Instrumentalness, Valence. The Recall, ROC AUC and Precision are all
equal to 1.


We use the classifier to predict the class of the songs from the most similar playlist defined in the CF
step. We obtain 26 songs positively classified with a probability equal or higher than 75%.

The list of songs was sent to the user who was asked to rate the recommendations on the three
following points:

**Personal preference** : on a scale from 1 to 5, how much did the user like the track.

**Musical Coherence** : on a scale from 1 to 5, how coherent is the track with the seed playlist.

**Novelty** : 1 if the track is a newly discovered song and 0 if the user already knew it.

**Music** Joyhauser (^) - Entropy - [ Wex 10 ] Remix **Preference** 5 **Coherence** 5 **Novelty** 1
A*S*Y*S I Hate Models - The Acid - Eternal Loneliness- Original Version 54 55 10
UMEK Luca Agnelli - Managing the Moments- Oceanus 35 45 10
Anthony Rother Traumer - Azure - Brahmall (^34  11  10)
Mark Greene Laherte - Virtus- Whats Good 54 54 11
Alberto Ruiz Alex Metric -- The Q Don't Start 42 51 11
Kenneth Bager Chaty - Stella Path - Drums Of Steel- Rafael Cerato Remix (^55  32  10)
Spektre Sugar - Heart Less- Carnival of Souls - SAMA Remix (^33  45  11)
Ahzee Sebastian Mullaert - Born Again (Short Radio Edit)- You're An Orchestra In The Cosmos (^) - 3 3 1
Joris Voorn RemixHelen&Boys - About Us (^) - Original Mix 54 13 01
In Verruf Der Dritte Raum - Too Much DMT- Hale Bopp (^) - Maceo Plex Edit 53 53 01
Illnurse Saive - Zibal- All Day - Original Mix (^45  54  11)
Massano Pete Tong - (^) - Velocita' Placebo -- Original mixOriginal 44 34 11
Matt Morra - Resort To Violence 5 5 1
**Average** (^) Fig 20. Ratings of the recommended songs **4,08** (^) **3,64 76%**
We obtain 76% of newly discovered songs, with an average score of 4.08 in terms of likeability and
3.64 for the coherence.


### 3 .7 Limits of our model and further improvements

#### 3 .7.1 Limits of the existing model

Through the analysis of the results we could determine two main points that could be further
improved:

- **The size of the data for CF** : We encountered the sparsity problem when running our CF model. The
reason is because the size of the dataset was not big enough. The playlist-song matrix was too sparse
because the users did not have enough songs in common. This can be solved by having a greater pool
of users and their playlists.
- **The sample data for CBF** : Sampling the data to train the KNN classifier biased the results because
we chose the sample from the same dataset where we later used the model to predict the songs.

Ideally, the songs from the sample data should be excluded from the recommendation dataset.

#### 3 .7.2 Additional improvements...........................................................................................

Additional layers can be added to our RS to improve the quality, precision, and coherence of the
recommendations:

- **User feedback** : We could use the rating system developed to score the example on the whole
recommendation set, by asking each user to give their feedback. The ratings could then be used to
construct more robust CF and CBF predictive models.
- **NLP** : We observed several resemblances in the names of the similar playlists and the songs
contained in them. For example, the most similar playlist of _Hard Techno_ is _Techno Electro._

We can use a similar NLP model to that of Spotify, by converting the titles of the songs and playlists
to word vectors and constructing a matrix to map the similarities between the playlists. The most
similar ones being the ones that contain the most words in common.


Beyond the technical limits, their business implications, and the domain specific challenges that they
face, RS also have an ethical impact both on the users and the artists.


## Conclusion

Hybrid models offer solutions to the issues faced by MRS from a technical perspective as well as the
challenges related to the nature of music as a form of entertainment. Using a community-based filter
on top allows to narrow the pool of recommendations making them more precise and coherent.
However, it is essential that the ethical questions raised by RS have more attention in upcoming
research.


## References

```
1) Shardanand, U. and Maes, P. (1995). Social Information Filtering: Algorithms for Automating
“Word of Mouth”, in CHI ’95, pp. 210–217.
```
##### 2) JPANNACH RACTICAL D.,CHALLENGES AND BONNIN G., AND AKPPLICATIONSAMEHKHOSH I. (2018). MUSIC RECOMMENDATIONS: ALGORITHMS,

```
3) Johnson, C. net/MrChrisJohnson/algorithmic(2014). Algorithmic Music Discovery at Spotify, Online https://de.slideshare. -music-recommendations-at-spotify.
```
```
4) Schedl M., Zamani H., Chen Cin music recommender systems research. In: International Journal of Multimedia Information -W., Deldjoo Y.,^ Elahi M. (2017) Current challenges and visions
Retrieval (2018) 7:95– 116
```
```
5) Johnson, C. and Newett, E. (2014). From Idea to Execution: Spotify’s Discover Weekly, Online
https://de.slideshare.net/MrChrisJohnson/ from-idea-to-execution-spotifys-discover-
weekly/12-Insight_users_spending_ more_time.
6) Schedl M, Knees P, Gouyon F (2017) New paths in music recommender systems research. In:
Proceedings of the 11th ACM conference on recommender systems (RecSys 2017), Como, Italy
```
```
7) https://newsroom.spotify.com/
```
```
8) https://www.comparitech.com/blog/vpnfigures/#:~:text=Netflix%20has%20over%2013%2C900%20titles,libraries%20as%20of%20Apr-privacy/netflix-statistics-facts-
il%202020.
```
9)^ https://www.quora.com/How-many-books-does-Amazon-have-for-sale^


10) Schäfer T, Sedlmeier P, Stdtler C, Huron D (2013) The psycho- logical functions of music
listening. Front Psychol 4(511):1– 34

11) Schmetaäfer T, Mehlhorn-analysis. Personal Individ Differ 116:265 C (2017) Can personality traits predict musi–273. - cal style preferences? A
https://doi.org/10.1016/j.paid.2017.04.061

12) Schäfer T, Auerswald F, Bajorat IK, Ergemlidze N, Frille K, Gehrigk J, Gusakova A, Kaiser B,
Pfeedätzold RA, Sanahuja A, Sari S, Schramm A, Walter C, Wilker T (2016) The effect of social - back on music preference. Musicae Sci 20(2):263–268. https://
doi.org/10.1177/1029864915622054

13) - Gross J (2007) Emotion regulation: conceptual and empirical foundations. In: Gross J (ed)
Handbook of emotion regulation, 2nd edn. The Guilford Press, New York, pp 1– 19

14) Boer D, Fischer R (2010) Towards a holistic model of functions of music listening across cultures: a culturally decentered qualitative approach. Psychol Music 40(2):179– 200

15) Punkanen M, Eerola T, Erkkilperception of emotions in music by depressed patients. J Affect Disord 130(1ä J (2011) Biased emotional recognition in depression: –2):118– 126

16) Barthet M, Fazekas G, Sandler M (2012) Multidisciplinary perspectives on music emotion
recognition: Implications for content and contextinternational symposium on computer music modelling and retrieval, pp 492-based models. In: Proceedings of – 507

17) Dey AK (2001) Understanding and using context. Pers Ubiquitous Comput 5(1):4–7.
https://doi.org/10.1007/s007790170019

18) Gillhofer M, Schedl M (2015) Iron maiden while jogging, debussy for dinner? An analysis of music listening behavior in context. In: Proceedings of the 21st international conference on
multimedia modeling (MMM), Sydney, Australia


19) Kaminskas M, Ricci F, Schedl M (2013) Locationtagging and hybrid matching. In: Proceedings of the 7th ACM conference on re-aware music recommendation using autocommender -
systems (RecSys), Hong Kong, China

20) Cebritemporal context awareness. In: Proán T, Planagumà M, Villegas P, Amatriain X (2010) Music recommendations with - ceedings of the 4th ACM conference on recommender
systems (RecSys), Barcelona, Spain

21) Pettijohn T, Williams G, Carter T (2010) Music for the seasons: seasonal music preferences in college students. Curr Psychol 29(4):328– 345

22) Gillhofer M, Schedl M (2015) Iron mamusic listening behavior in context. In: Proceedings of the 21st international conference on iden while jogging, debussy for dinner? An analysis of
multimedia modeling (MMM), Sydney, Australia

23) Cunningham SJ, Nichols DM (2009) Exploring social music behavmusic selection at parties. In: Proceedings of the 10th international society for music iour: an investigation of
information retrieval conference (ISMIR 2009), Kobe, Japan

24) Celma, O. and Cano, P. (2008). From hits to niches?: Or how popular artists can brecommendation and discovery, in NETFLIX ’08, pp. 5:1–5:8. ias music

25) Steck, H., von Zwol, R., and Johnson, C. (2015). Interactive Recommender Systems, Onlinehttps://de.slideshare.net/MrChrisJohnson/ interactive-recommender-systems-with-netflix (^) -
and-spotify.
26) Lamere, P. and Celma, O. (2011). Music Recommendation and Discovery Remastered, Tutorial at ACM RecSys 2011, Online https://musicmachinery.com/2011/10/24/ music-
recommendation - and-discovery-remastered-a-tutorial/.
27) Kaminskas M, Ricci F (2012) Contextual music information retrieval and recommendation: state of the art and challenges. Comput Sci Rev 6(2):89– 119
28) Dror G, Koenigstein N, Koren Y, Weimer M (2011) The yahoo! music dataset and kddIn: Proceedings of the 2011 international conference on KDD Cup 2011, vol 18, pp 3–-18. cup’11.
JMLR.org


29) Garfinkel, Robert & Gopal, Ram & Pathak, Bhavik & Venkatesan, Rajkumar & Yin, Fang.
(2006). Empirical Analysis of the Business Value of Journal. 10.2139/ssrn.958770. Recommender Systems. SSRN Electronic

30) Jannach, Dietmar & Jugovac, Michael. (2019). Measuring the Business Value of
Recommender Systems.

31) Ricci, F., Rokach, L. and Shapira, B., 2015. Recommender Systems Handbook.

32) Lops, Pasquale & de Gemmis, Marco & Semeraro, Giovanni. (2011). Content-based
Recommender Systems: State of the Art and Trends. 10.1007/978- 0 - 387 - 85820 - 3_3.

33) Schedl, Markus & Knees, P. & McFee, B. & Bogdanov, Dmitry & Kaminskas, Marius. (2015).
Music recommender systems. 10.1007/978- 1 - 4899 - 7637 - 13.

34) Manning, C.: Introduction to Information Retrieval. Cambridge University Press, Cambridge
(2008)

35) Downie, J. Stephen. Music information retrieval (Chapter 7). In Annual Review of Information
Science and Technology 37, ed. Blaise Cronin, 295-340. Medford, NJ: Information Today,

2003. Available from [http://music-ir.org/downie_mir_arist37.pdf](http://music-ir.org/downie_mir_arist37.pdf)

36) Pachet, F.: Knowledge management and musical metadata. Idea Group (2005)

37) Bogdanov, D., Herrera, P.: Taking advantage of editorial metadata to recommend music. In:
Int. Symp. on Computer Music Modeling and Retrieval (CMMR’12) (2012).

38) Haupt, J.: Last.fm: People-powered online radio. Music Reference Services Quarterly 12(1),
23 – 24 (2009).

39) Turnbull, D.R., Barrington, L., Lanckriet, G., Yazdani, M.: Combining audio content and social
context for semantic music discovery. In: Proceedings of the 32nd international ACM SIGIR
conference on Research and development in information retrieval, pp. 387–394. ACM (2009)


40) Pazzani, M., Billsus, D.: Learning and revising user profiles: The identification of interesting
web sites. Machine Learning 27(3), 313–331 (1997).

41) Casey, M.A., Veltkamp, R., Goto, M., Leman, M., Rhodes, C., Slaney, M.: Content-based music
information retrieval: Current directions and future challenges. Proceedings of the IEEE
96(4), 668–696 (2008)

42) Tzanetakis, G.; Cook, P. Musical genre classification of audio signals. IEEE Trans. Speech Audio
Process. 2002, 10, 293–302.
43) H. C. Chen, A. L.P. Chen, 2001. “A Music Recommendation System Based on Music Data
grouping and User Interests”. Proc. Of ACM International Conference on Information and
Knowledge Management CIKM’01.

44) Wang, X., Rosenblum, D., Wang, Y.: Context-aware mobile music recommendation for daily
activities. In: Proceedings of the 20th ACM international conference on Multimedia, pp. 99–

108. ACM (2012)

45) Goldberg, D., Nichols, D., Oki, B., Terry, D.: Using Collaborative Filtering to Weave an
Information Tapestry. Communications of the ACM 35(12), 61–70 (1992). URL [http:](http:)
//www.xerox.com/PARC/dlbx/tapestry-papers/TN44.ps. Special Issue on Information
Filtering

46) Resnick, P., Iacovou, N., Suchak, M., Bergstrom, P., Riedl, J.: GroupLens: An Open
Architecture for Collaborative Filtering of Netnews. In: Proceedings of ACM 1994 Conference
104 Pasquale Lops, Marco de Gemmis and Giovanni Semeraro on Computer Supported
Cooperative Work, pp. 175–186. ACM, Chapel Hill, North Carolina (1994). URL
citeseer.ist.psu.edu/resnick94grouplens.html

47) D.W. Oard and J. Kim, “Implicit Feedback for Recommender Systems”, Proc. 5th DELOS
Workshop on Filtering and Collaborative Filtering, pp. 31–36, 1998.

48) Hu, Yifan & Koren, Yehuda & Volinsky, Chris. (2008). Collaborative Filtering for Implicit
Feedback Datasets. Proceedings - IEEE International Conference on Data Mining, ICDM. 263-

272. 10.1109/ICDM.2008.22

49) Bobadilla, Jesus & Ortega, Fernando & Hernando, A. & Gutiérrez, A.. (2013). Recommender
systems survey. Knowledge-Based Systems. 46. 109–132. 10.1016/j.knosys.2013.03.012.

50) Green, S.J., Lamere, P., Alexander, J., Maillet, F., Kirk, S., Holt, J., Bourque, J., Mak, X.W.:
Generating transparent, steerable recommendations from textual descriptions of items. In:
ACM Conf. on Recommender Systems (RecSys’09), pp. 281–284 (2009)

51) Schafer, J.B., Frankowski, D., Herlocker, J., Sen, S.: Collaborative filtering recommender
systems. In: The Adaptive Web, pp. 9–10. Springer Berlin / Heidelberg (2007)


52) Ricci F., Rokach L., Shapira B. (2011) Introduction to Recommender Systems Handbook. In:
Ricci F., Rokach L., Shapira B., Kantor P. (eds) Recommender Systems Handbook. Springer,
Boston, MA. https://doi.org/10.1007/978- 0 - 387 - 85820 - 3_1

53) Desrosiers, Christian & Karypis, George. (2011). A Comprehensive Survey of Neighborhood-
Based Recommendation Methods. 10.1007/978- 0 - 387 - 85820 - 3_4.

54) Burke, Robin. (2002). Hybrid Recommender Systems: Survey and Experiments. User
Modelling and User-Adapted Interaction. 12. 10.1023/A:1021240730564.
55) Chris Johnson, Spotify Discover Weekly: From idea to execution, 2015.
https://www.slideshare.net/MrChrisJohnson/from-idea-to-execution-spotifys-discover-
weekly/31-1_0_0_0_1

56) Medium article: https://medium.com/s/story/spotifys-discover-weekly-how-machine-
learning-finds-your-new-music-19a41ab76efe

57) O’Dair M., Fry A. (2018) Beyond the black box in music streaming: the impact of
recommendation systems upon artists, pp 65- 77

58) Wikström, P. (2013), The music industry: music in the cloud, second edition. Cambridge:
Polity.

59) Gillespie, T. (2014), The relevance of algorithms. In T Gillespie, P J Boczkowski and K A Foo
(eds), Media technologies: Essays on communication, materiality, and society. (pp. 167-194).
Cambridge, Mass: MIT.

60) Pasquale, F. (2015), Black box society: the secret algorithms that control money and
information. Cambridge, Mass: Harvard University Press.

61) Srnicek, N. (2016), Platform capitalism. Cambridge: Polity.

62) Bucher, T. (2018), If... then: Algorithmic power and politics. Oxford: Oxford University Press.

63) Cush, A. (2018), Spotify Has No Idea What It’s Doing With XXXTentacion, Spin, 19 June.
Available online: https://www.spin.com/2018/06/spotify-xxxtentacion-playlists-death


64) Beuscart J-S., Coavoux S., and Maillard S. (2019), Music recommendation algorithms: What
influence do they have on what users listen to? https://hellofuture.orange.com/en/music-
recommendation-algorithms-what-influence-do-they-have-on-what-users-listen-to/

65) Adomavicius, Gediminas & Bockstedt, Jesse & Curley, Shawn & Zhang, Jingjing. (2013). Do
Recommender Systems Manipulate Consumer Preferences? A Study of Anchoring Effects.
SSRN Electronic Journal. 24. 10.2139/ssrn.2285042.

66) Eli Pariser, The filter bubble, Penguin Books, 2011
