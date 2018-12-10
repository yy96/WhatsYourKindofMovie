## 1. Introduction of the Project
Our project will be focusing on the movie industry. Based on the dataset obtained from MovieLens, we aim **to investigate the preference of the audience and the characteristics of commonly rated good movies and bad movies.** 

Here to find out more though a short video. (**Check out here!**)
[![Check Out the Video!](https://j.gifs.com/4RgkL2.gif)](https://www.youtube.com/watch?v=XTxMXeBN4kg&t=)

## 2. Data Preparation 
- **Raw Datasets**
<br>
We will be using the available data sets due September 26, 2018 from the GroupLens website. It contains *27753444* ratings and *1108997* tag applications across *58098* movies. These data were created by *283228* users between January 09, 1995 and September 26, 2018. The total size of the datasets is *265MB*. It consists of 6 csv files: ratings.csv, tags.csv, movies.csv, links.csv, genome-scores.csv and genome-tags.csv.

<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        }
</style>

Name of csv file | Size (KB) | Number of Rows | Content
---------------- | --------- | -------------- | -------
Movies | 968 | 58098 | movieId: ID of a movie <br> Title: title of a movie from themoviedb.org <br> genres: genres which the movie belongs to
Ratings | 741407 | 27753444 | userId: ID of the user <br> movieId: ID of the movie referenced from Movies dataset <br> rating: A number given by a user on a 5-star scale, with half-star increments <br> timestamp: Seconds since midnight Coordinated Universal Time (UTC) of January 1, 1970.
Tags | 38814 | 1108997 | userId: ID of the user <br>  movieId: ID of the movie <br> tag: A single word or short phrase description about the movie given by the user. <br> timestamp: Seconds since midnight Coordinated Universal Time (UTC) of January 1, 1970.
Links | 1238 | 58098 | movieId: identifier for movies used by [Movie Lens](https://movielens.org) <br> imdbId: identifier for movies used by [imdb](http://www.imdb.com) <br> tmdbId: identifier for movies used by [tmdb](https://www.themoviedb.org)
Genome-Scores | 405129 | 14862528 | movieId: identifier of the movie <br> tagId: identifier of the tag, referenced from Genome-Tags dataset <br> relevance: Scores of the movie in a particular genre (how strong it exerts this kind of content in the movie)
Genome-Tags | 18 | 1128 | tagId: identifier of the tag <br> tag: name of each tag

{: .tablelines}
<br>
- **Cleaned Datasets**
<br>
The data retrieved from the websites are in separate files. For the easy manipulation of data for network analysis and text analysis. We have merged and joined the datasets based on their common keys and only kept the relevant information that is needed for the analysis. After the clean-up, there are 2 master table for the further analysis

Name of csv file | Source Table | Content
---------------- | ------------ | -------
Movie Master | Links <br> Movies | **movieId: primary key** <br> Title <br> genres <br> imdbId <br> tmdbId
Review Master | Ratings <br> Tags | **userId_movieId: primary key** <br> userId <br> movieId <br> rating_combined: ratings for the user and the particular movie. If thereare multiple entries for the uerId_movieId, an average value is calculated and stored <br> timestamp_combined_ratings: If there are multiple entries for the userId_movieId, the earliest one is stored <br> tag_combined: stores all the tags for the user for this particular movie <br> timestamp_combined_tags: If there are multiple entries for the userId_movieId, the earliest one is stored


## 3. Network Analysis

### 3.1 Netowrk Building
The movie review network is constructed based on the following rules:
```
Nodes - people who write the review for the movie 
Edges - people who watch/rate the same movies
Node Size - number of movies/ratings the person give
Edge weights - number of movies that both node users watched in common
```
The network is created as shown below

![Image](pictures/overall_network.png)

### 3.2 Network Attributes

#### 3.2.1 Sombe Basic Stats for the Network

<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        }
</style>
--------------- | ----
Number of nodes | 2003
Number of edges | 232910
Average degree | 232.5612
Average shortest path distance | 1.9574773703580493
Number of connected components | 53

{: .tablelines}

<br>
Degree Distribution plots:
![Image](pictures/combined_degree_distribution.png)
As seen from the loglog plot, the general shape of the degree distribution of the movie review network follows a power law where a large number of nodes have small connections and small number of nodes have large connctions.

#### 3.2.2 Centrality Measures

- **Degree Centrality**
> Degree centrality is the most basic method of defining centrality, basing the centrality 
> only on the number of neighbours a node has.

The top 3 values for degree centrality are as shown below. The top degree centrality is rather high this suggests they are highly connected to other nodes in the network. Thus, these nodes are most likely to be clustered in the center of the network.
<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        }
</style>
1 | 2 | 3
------------------ | ------------------ | ----------------- 
0.9125874125874126 | 0.7987012987012987 | 0.7562437562437563
{: .tablelines}
<br>
- **Betweenness Centrality**
> Betweenness centrality quantifies the number of times a node acts as a bridge along the 
> shortest path between two other nodes.

The top 3 values for betweeness centrality are as shown below. The max value is 0.085 which is close to zero. Small betweenness centrality means users are generally connected directly to each other as there are very few times that the node is acting as a bridge.
<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        }
</style>
1 | 2 | 3
------------------- | -------------------- | -------------------- 
0.08480222837203849 | 0.038790113817311976 | 0.027062225532841173 
{: .tablelines}
<br>
- **Eigenvector centrality**
> The eigenvector centrality thesis read: A node is important if it is linked to by other 
> important nodes. It is a measure of the influence of a node in a network.

The top 3 values for eigenvector centrality are as shown below. The top eigenvector centrality scores are close to zero which implies that users do not have much influence on one another even though they are top ranked based on eigenvector centrality.
<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        }
</style>
1 | 2 | 3
------------------- | -------------------- | --------------------  
0.06192699109472035 | 0.06107418624610914 | 0.06015765571794681 
{: .tablelines}

#### 3.2.3 Exploring Potential Correlations in Nodes' Attributes
In this section we are aiming to the potential correlation by assigning relevant information as the attributes for the nodes. We are interested to find out the relationship between the active-ness of an user and the average ratings/sentiment scores he/she gives.The defintion for the terms used are as follow:

```
Active-ness - It is defined by the number of distinct movies a particular user reviewed.
Average Ratings - It is the average ratings results for all the ratings the user has given.
Average Sentiment Scores - Sentiment scores are calculated based on the tags given by the user. 
This is the average results for the sentiment scores derived from all the tags the user has given.
```

We will approach the problem by building *Rating Netwok* and *Sentiment Score Network*. Beisdes the rules for the general network, the additional features for these new networks will be:
```
Node Size - Number of distinct movies the user watched
Node Colour - The average ratings/sentiment scores the user give based onthe colour gradient
```

- **Ratings Network**
We have constructed the network based on the rules mentioned above. The network is as shown below. As it is a pretty dense network, we plotted a scatter plot to investigate the relationship further. 

![Network](pictures/Ratings_Network.png)

Due to the densely populated graph, the relationship between the node size and node colour is not clear. With the additional scatter plot, we hope to get a clearer picture. The scatter plot can be roughly divided into 3 sections, the vertical line at the front, slanted line with steep graident in the middle and horizontal line at the end. Both vertical line and horizontal line indicate there is no correlation between the acive-ness of an user and the average ratings he/she gives, while the steep gradient for the slanted line indicates weak correlation. However, the the percentage of points fall in the middle section (slanted line with steep gradient) is rather small compared to other 2 sections. Thus, we would conclude there is very weak correlation between the active-ness of an user and the average ratings he/she gives.

- **Sentiment Score Network**

### 3.3 Community Detection
![Image](pictures/community_detection.png)

While this is a rather dense network , the nodes belong to the same community are closely clustered together with quite clear separation between each cluster. In the following section, we will move on to explore more about the common traits and distinct characteristics of each section.

Download dataset by putting the link in Github

http://nbviewer.jupyter.org/github/yy96/WhatsYourKindofMovie/blob/master/PythonNotebook/Sentiment%20Analysis%20on%20Tags.ipynb
