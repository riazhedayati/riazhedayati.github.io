---
layout: post
title: Can a song's lyrics predict it's genre? (part 1)
subtitle: Web scraping with Beautiful Soup in Python
---

We've all heard the saying "don't judge a book by it's cover." Today we'll try to see if the same holds true for music: Can we judge a song's genre by it's lyrics?

## Defining our target
If we’re going to try to predict a song’s genre using its lyrics, one big issue to think about first is the subjective nature of our target variable, _genre_. For example, is Taylor Swift a pop artist, or a country singer? Similarly, are _Alternative_ and _Heavy Metal_ separate genres, or can you just classify both as just _Rock_?  Classifying a song within a genre is a subjective exercise.
 
Luckily, I was able to avoid listening to hundreds of songs and making my own subjective assessments on genre, thanks to a website called [Songlyrics](http://www.songlyrics.com/news/top-genres/country-music). Songlyrics.com provided lyrics for the top 100 songs in 6 different genres: _Christian_, _Country_, _Rap_, _Pop_, _R&B_, and _Rock_. While there was no information from Songlyrics on how they made their decisions (How did they determine genre? Top 100 songs by what metric?), the songs passed the sniff test: Hank Williams was in the Country genre, and Guns N’Roses in the Rock genre.

## Gathering the data
Now that we have a data source, we need to actually capture that data. There were 100 songs in 6 different genres, so our dataset will have 600 rows, where each row is a song. To begin, our dataset will have 3 columns: songinfo (including artist name and song title), lyrics, and genre.

To create the dataset, I wrote a small python program using the [beautiful soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) library to scrape the songinfo, lyrics, and genre for each of the 600 songs. Beautiful Soup is a handy library that will allow you to easily parse HTML tags on websites, allowing you to more easily capture the information you want.

### Web Scraping with Beautiful Soup
First, I imported the relevant libraries and created 3 empty vectors, one for each column of desired data to be stored. I also created a list containing 6 urls, one for each of the genres.

<pre><code class="language-python line-numbers">from bs4 import BeautifulSoup
import urllib2
import numpy
import csv
 
lyricsvector = [] #input (bag of words)
genrevector = [] #target
songinfovector = []  #metadata (artist and songname)

# List the URLs here
urllist = [
"http://www.songlyrics.com/news/top-genres/christian/",
"http://www.songlyrics.com/news/top-genres/country-music/",
"http://www.songlyrics.com/news/top-genres/hip-hop-rap/",
"http://www.songlyrics.com/news/top-genres/rhythm-blues/",
"http://www.songlyrics.com/news/top-genres/pop/",
"http://www.songlyrics.com/news/top-genres/rock/"]
</code></pre>

Next, I looped through each genre in the urllist and used beautiful soup to parse the html, looking for links to the top 100 songs in each genre. The urls for each song in the top 100 were stored in a new list called songlinks. 

<pre><code class="language-python line-numbers"># convert top 100 urls into parseable html
for i in range(0,6):
    url = urllib2.urlopen( urllist[i] )
    doc = url.read()
    soup = BeautifulSoup(doc, 'html.parser')
    div = soup.find( 'div', { 'class': 'box listbox' } )
 
# get genres
    title = soup.title.get_text().encode('ascii', 'ignore').split(' ')
    index100 = title.index('100')
    indexSongs = title.index('Songs')
    genre = ' '.join(title[(index100+1):(indexSongs)]).encode('utf-8')
 
# create list of top 100 songs by genre
    print genre
    songs = div.find_all('a')
    songlinks = []

# create loop to extract song links
    for j in range(0,200): #[0::2]:
        songlink = songs[j].get('href').encode('ascii', 'ignore')
        songlinks.append(songlink) #output links to a list called songlinks

    songlinks = filter(None, songlinks)
    songlinks = [songlink for songlink in songlinks if (len(songlink.split('/'))==6)]
</code></pre>

Now that we have the song specific links, it’s time to extract the lyric text from each of those pages, along with some other relevant information. A typical page looks something like [this](http://www.songlyrics.com/the-beatles/yesterday-lyrics/). 

Using the beautiful soup library again, we are able to capture the lyrics, genre, and songinfo from each url, and save the results to the empty vectors created earlier. 
 
<pre><code class="language-python line-numbers"># loop through songlinks list to get the actual lyrics
    for k in range(0,len(songlinks)):
        songurl = urllib2.urlopen( songlinks[k] )
        songdoc = songurl.read()
        songsoup = BeautifulSoup(songdoc, 'html.parser')
        songinfo = songsoup.title.get_text().encode('ascii', 'ignore')
        print songinfo, 'is number', k

        songdiv = songsoup.find( 'div', { 'id': 'songLyricsDiv-outer' } )
        lyrics = songdiv.getText().replace("\n", " ").replace("\'", "").replace("\r", " ").encode('utf-8')

        lyricsvector.append(lyrics)
        genrevector.append(genre)
        songinfovector.append(songinfo)
 
</code></pre>

After looping through all of the links, data was exported to a csv, which you can find [here](/img/songlyrics/textminingAllLyrics.csv).
Now that we have a dataset, it is time to do some cleaning and exploration. Please see [part 2 of this post](https://riazhedayati.github.io/blog/predict-song-genre-pt2/) for the next steps.

