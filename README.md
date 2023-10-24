![image](https://storage.googleapis.com/pr-newsroom-wp/1/2018/11/Spotify_Logo_CMYK_Green.png)
<br/>
<br/>
<br/>

# Spotify EDA and Clustering

### **Statement of Work:** 
*This was a 7-week project for BA780, a course in the MSBA Program at Boston University, Questrom School of Business. The work completed her was done by a team of 6 (credited in the notebook). My main contributions were being a PM for the team, as well as performing ETL and working with the Spotify API to further build out the dataset.* 

### **What's in Here:** 
In the ``README`` file, I'll provide a condensed version of the notebook and report the main findings and the EDA at a high level. Additionally, there are some ``plotly`` charts that are not visible in the notebook. To work around this issue, I'll paste the graphs relevant to our analysis in here.

## Problem Statement
The goal of this project was to try and understand *'What makes a song popular?'*. To do this we leveraged ``Track Popularity`` from the [Spotify API](https://developer.spotify.com/documentation/web-api).

Per Spotify Documentation, here's the definition of ``Track Popularity``
- The popularity of the track. The value will be between 0 and 100, with 100 being the most popular.
The popularity of a track is a value between 0 and 100, with 100 being the most popular. The popularity is calculated by algorithm and is based, in the most part, on the total number of plays the track has had and how recent those plays are.
Generally speaking, songs that are being played a lot now will have a higher popularity than songs that were played a lot in the past. Duplicate tracks (e.g. the same track from a single and an album) are rated independently. Artist and album popularity is derived mathematically from track popularity. Note: the popularity value may lag actual popularity by a few days: the value is not updated in real time.

*Another thing to keep in mind is that because ``Track Popularity`` is a live metric, the scope of the analysis is limited to what's making tracks popular right now.*

With this information, scope, and our goal in mind, we wanted to ask: **How can Track Popularity be utilized to create value for artists and Spotify stakeholders?**.

## API Work

### Description 
Before jumping into the analysis on a high-level, I wanted to touch on the API work done here. It was a good challenge to learn how to work with a new API and more specifically try and think in the mindset of an App Developer. Since the only way to work with the API is to setup an 'app', you have to be really considerate about how much you're calling the API and consider things such as batching, even if your goal is just to collect data on the backend. 

### Getting Started
Before you can start querying any data, Spotify requires a few things of you when getting started.

**Prerequisites:**
- Set up a developer account on [Spotify for Developers](https://developer.spotify.com)
- Create your first app and retrive a ``client id`` and ``client secret``

Once this is complete, you're ready to retrive your API token. From here, its just a matter of formatting your query for the data you want to retrive and including your token in that request.

*Note that tokens auto-expire every 60 minutes after they're created, so this will require you to continue to rerequest a new token inside of whatever your program is doing*

### Making Requests (Psuedocode)
To make things easier for people like us, theres a nice project called [Spotipy](https://spotipy.readthedocs.io/en/2.22.1/). It allows you to instantiate an request object with your token, and has methods corresponding to all the types of requests you can make to the Spotify API. It's a really nice tool that saves lots of time, but will still require you to understand the API in and out.

For our project, we focused on three queries: ``get_trakcs()``, ``get_artists()`` and ``get_playlist()``. We wanted to add existing information to data we already had on a Kaggle Dataset of about ~20,000 Spotify tracks. Additionally, we wanted to collect data from a blend playlist that we created as a team, just for fun :). 

**Here's How it Works:**
- Each Spotify track has a unique identifier called ``URI``
  - ``(ex: spotify:track:3AIBCyWVRvGUuTkeszbi12)``
- For each URI in our dataset, we create a list of 50 ``URIs`` and pass them to ``get_tracks()``
- ``get_tracks()`` takes the 50 tracks each iteration and will return a JSON object of all the data we're looking for that is related to a track.
- The returned JSON object from ``get_tracks()`` contains an ``Artist URI``, which has some extra information on the specific Artist for a track
- We can refactor to ``get_artists()`` and repeat the process for ``get_tracks()`` but with our obtained ``Artist URis``
- Finally, we convert all the retrived JSON objects into a pandas DataFrame

The nature of ``get_playlist`` is similar to what is described above. However, instead of passing individual ``URIs``, the playlist itself has a ``Playlist URI`` which we can just pass once to get the necessary data.

This is just a simplified explanation of what is actually going on. Feel free to dig into [``add_columns.ipynb``](https://github.com/jbblancojr/Spotify_EDA_and_Clustering/blob/main/add_columns.ipynb) and [``get_playlist.ipynb``](https://github.com/jbblancojr/Spotify_EDA_and_Clustering/blob/main/get_playlist.ipynb) to see what is actually required to complete this task.

## Dataset
After the API work is finished, we merge it with the existing Kaggle dataset and we're ready to get going. Due to the nature of JSON objects, our dataset is semi-structured in nature and will require a fair amount of cleaning and transformations to derive any meaningful insights. In the [Jupyter Notebook](https://github.com/jbblancojr/Spotify_EDA_and_Clustering/blob/main/A01_Decoding_the_Secret_to_Popularity_Spotify.ipynb), you'll find an extensive dive into the dataset and the necesary steps it took to do so, but I'll spare the details here.

## EDA Main Findings
There is a lot of summarization and distributions in the first half of the notebook to better understand individual varibles within the dataset. The true analysis starts in the latter half. 

### Normalizing ``Track Popularity``
When we checked the distribution for ``Track Popularity``, we found a right skew in the data. There are a lot of outliers on the unpopular side of things. Something unique about Spotify, is that pretty much anyone can upload a song. This is great in concept and in practice, but for our analysis it does make the data a little more noisy. Sometimes users on Spotify take advantage of this to post unreleased songs by real artists, or just to upload their own projects that aren't necesarilly production level. These are likely the tracks that represent the outliers, so it makes sense to just remove them altogether. We utilzed IQR to do this.



### Key Variables of Interest (Restating the Business Problem)
Referring back to the Business Problem, we want to try and figure out what variables in the dataset are driving ``Track Popularity``. When we pulled the track object from the Spotify API, we got a bunch of other useful information. Each track contains specific audio features (ex. ``Danceability`` or ``Energy``) that we can use to analyze relationships with in terms of ``Track Popularity``.



## Clustering
