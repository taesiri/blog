---
layout: post
title: "Mining Twitter for Training Language Models [DRAFT]"
author: "Mohammad Reza Taesiri"
categories: nlp
tags: [NLP, Language Models, Twitter, Social]
image: twitter-mining/heading.jpg
---

<link rel="stylesheet" href="../assets/css/table.css">
<script src="https://d3js.org/d3.v3.js"></script>
<style>
      rect.bordered {
        stroke: #E6E6E6;
        stroke-width:2px;   
      }

      text.mono {
        font-size: 9pt;
        font-family: Consolas, courier;
        fill: #aaa;
      }

      text.axis-workweek {
        fill: #000;
      }

      text.axis-worktime {
        fill: #000;
      }
</style>

In the last couple of days, One of my friends from UCI and I  have started a new project, a very different kind to my taste! I have pretty much zero knowledge of natural language processing (NLP) and I wanted to change that once for all. So I joined my friend in doing some empirical study of social networks.

There are lots of courses and books about modern NLP powered by neural networks, but almost all of them are for the English language. (I'd better say, they are not for my native language). So I decided to collect my own database of Persian text and start doing experiments on them. Twitter is an amazing source of such texts, and more importantly, with official API, mining it is super easy and easy and convenient.

## Capturing Stream of Tweets with Tweepy

There are multiple Python libraries to interact with Twitter API, my choice is Tweepy (disclaimer: it was a random first choice). Setting it up is pretty easy, you only need to get ``consumer_key``, ``consumer_secret``, ``key`` and ``secret`` parameters from the developer's website of Twitter.

After getting those values, you can setup Tweepy like this:

----

```python
import sys
import tweepy as tw
import json
import io
import sys


auth = tw.OAuthHandler("consumer_key", "consumer_secret")
auth.set_access_token("key", "secret")

api = tw.API(auth, wait_on_rate_limit=True)
```

There is a Streaming API that allows us to automatically receive a detailed ``JSON`` object corresponding to each tweet satisfying our ``Query``.  After receiving the ``JSON`` object, you can push it into a database or like me, write a simple dump to disk function. We've collected more than 6 million tweets exactly like using the following code. It is simple and it works.

```python
class MyStreamListener(tw.StreamListener):
  def __init__(self, api):
    self.api = api
    self.counter = 0
    self.listOfTweets = []
    super(tw.StreamListener, self).__init__()

  def dump_on_disk(self):
    with open('tweets_stopwords_db.txt', 'a') as f:
      for t in self.listOfTweets:
        f.write(t + "\n\n")
  
    self.listOfTweets = []

  def on_status(self, status):
    self.counter += 1

    tj = json.dumps(status._json)
    self.listOfTweets.append(tj)

    if (len(self.listOfTweets)%2000 == 0):
      print("Dumping on Disk {}".format(self.counter))
      self.dump_on_disk()
```

Unfortunately, Twitter's API has one annoying limitation, you can't query all tweets in a particular language. To overcome this limitation, in our first run, we collected over 2 million tweets using a query containing 15 words (which we believed they are important). From these 2 million tweets, we filtered most commonly used words, which gave us this:

```python
words = ['به', 'از', 'که', 'در', 'این', 'را', 'با', 'است', 'رو', 'هم', 'برای', 'تو', 'ما']
```

Now in our second run, we queried for these most common words. Once in a while, we were receiving a network-related error. After digging [Tweepy's Github issues](https://github.com/tweepy/tweepy/issues/617#issuecomment-399180615), We ran Tweepy inside a *guarded loop*, which automatically resumes a crashed Stream.

```python
from ssl import SSLError
from requests.exceptions import Timeout, ConnectionError
from urllib3.exceptions import ReadTimeoutError

myStreamListener = MyStreamListener(api)
myStream = tw.Stream(auth = api.auth, listener=myStreamListener)

# create a zombie
while not myStream.running:
  try:
    # start stream synchronously
    print("Started listening to twitter stream...")
    myStream.filter(languages=['fa'], track=words)
  except (Timeout, SSLError, ReadTimeoutError, ConnectionError) as e:
    print("Network error occurred. Keep calm and carry on.", str(e))
  except Exception as e:
    print("Unexpected error!", e)
  finally:
    print("Stream has crashed. System will restart twitter stream!")
print("Somehow zombie has escaped...!")
```

----

## Do some plotting

Before jumping into the NLP part, and after collecting some data, It is always nice to plot some aspect of you collected data. The very first thing to come to mind is to plot a heatmap of twitter's activity in each hour. When doing this for all of your tweets, regardless of a tweet is an original tweet or a re-tweet, it gives an interesting chart about what hour people go to sleep.

<div id="dataset-picker" align='center'></div>
<br>
<div id="chart" align='center' ></div>

<script type="text/javascript">

      const url_template = "https://raw.githubusercontent.com/taesiri/looking_glass/master/data/WORD.csv"
      const words_list = ['ALL-STATS', 'ازدواج', 'بچه', 'تعداد', 'تولد', 'جمعیت', 'خارج', 'خانه', 'رشد', 'رفتن', 'زنان', 'فرزند', 'فقر', 'مالی', 'مهاجرت', 'پولدار', 'کار', ]

      function word_to_url(word_to_load) {
        return url_template.replace('WORD', word_to_load)
      }

      var margin = { top: 25, right: 0, bottom: 50, left: 0 },
          width = 615 - margin.left - margin.right,
          height = 900  - margin.top - margin.bottom,
          gridSize = Math.floor(width / 18),
          legendElementWidth = gridSize*1.55,
          buckets = 9,
          colors = ['#ffffe5', '#fff8c4', '#feeba2', '#fed676', '#febb4a', '#fb9a2c', '#ee7919', '#d85b0a', '#b74304', '#8f3204', '#8f3204'], // alternatively colorbrewer.YlGnBu[9]
          days = ["We", "Th", "Fr", "Sa", "Su", "Mo", "Tu", "We"],
          times = ["12A", "1A", "2A", "3A", "4A", "5A", "6A", "7A", "8A", "9A", "10A", "11A", "12P", "1P", "2P", "3P", "4P", "5P", "6P", "7P", "8P", "9P", "10P", "11P"];
          datasets = ['All-Stats', 'Jameh'];

      var svg = d3.select("#chart").append("svg")
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .append("g")
          .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      var dayLabels = svg.selectAll(".dayLabel")
          .data(days)
          .enter().append("text")
            .text(function (d) { return d; })
            .attr("x", function (d, i) { return 150 + (i * gridSize); })
            .attr("y", 0)
            .style("text-anchor", "middle")
            // .attr("transform", "translate(-6," + gridSize / 1.5 + ")")
            .attr("transform", "translate(" + gridSize / 2 + ", -6)")
            .attr("class", function (d, i) { return ((i >= 0 && i <= 4) ? "dayLabel mono axis axis-workweek" : "dayLabel mono axis"); });

      var timeLabels = svg.selectAll(".timeLabel")
          .data(times)
          .enter().append("text")
            .text(function(d) { return d; })
            .attr("x", 140)
            .attr("y", function(d, i) { return (i * gridSize); })
            .style("text-anchor", "middle")
            .attr("transform", "translate(-15," + gridSize / 1.5 + ")")
            .attr("class", function(d, i) { return ((i >= 7 && i <= 16) ? "timeLabel mono axis axis-worktime" : "timeLabel mono axis"); });

      var heatmapChart = function(csvFile) {
        d3.csv(csvFile,
        function(d) {
          return {
            day: +d.day,
            hour: +d.hour,
            value: +d.value
          };
        },
        function(error, data) {
          var colorScale = d3.scale.quantile()
              .domain([0, buckets - 1, d3.max(data, function (d) { return d.value; })])
              .range(colors);

          var cards = svg.selectAll(".hour")
              .data(data, function(d) {return d.day+':'+d.hour;});

          cards.append("title");

          cards.enter().append("rect")
              .attr("y", function(d) { return ( d.hour ) * gridSize; })
              .attr("x", function(d) { return  150 + ( d.day ) * gridSize; })
              .attr("ry", 4)
              .attr("rx", 4)
              .attr("class", "hour bordered")
              .attr("width", gridSize)
              .attr("height", gridSize)
              .style("fill", colors[0]);

          cards.transition().duration(1000)
              .style("fill", function(d) { return colorScale(d.value); });

          cards.select("title").text(function(d) { return d.value; });
          cards.exit().remove();

          var legend = svg.selectAll(".legend")
              .data([0].concat(colorScale.quantiles()), function(d) { return d; });

          legend.enter().append("g")
              .attr("class", "legend");

          legend.append("rect")
            .attr("x", function(d, i) { return legendElementWidth * i; })
            .attr("y", height)
            .attr("width", legendElementWidth)
            .attr("height",  gridSize / 2)
            .style("fill", function(d, i) { return colors[i]; });

          legend.append("text")
            .attr("class", "mono")
            .text(function(d) { return " ≥ " + Math.round(d); })
            .attr("x", function(d, i) { return legendElementWidth * i; })
            .attr("y", height + gridSize );

          legend.exit().remove();

        });  
      };

      heatmapChart(word_to_url(words_list[0]));

      var datasetpicker = d3.select("#dataset-picker").selectAll(".dataset-button")
        .data(words_list);

      datasetpicker.enter()
        .append("input")
        .attr("value", function(d){ return d })
        .attr("type", "button")
        .attr("class", "dataset-button")
        .on("click", function(d) {
          heatmapChart(word_to_url(d));
        });

  </script>

----

TBC ..

----

## Links

1. [Tweepy's Github Page](https://github.com/tweepy/tweepy)
1. [D3.js](https://d3js.org/)
1. [D3 Heatmap Tutorial](https://www.d3-graph-gallery.com/heatmap)
1. [D3 - Day / Hour Heatmap](http://bl.ocks.org/tjdecke/5558084)

## Image Credits

1. <a href="https://www.freepik.com/free-photos-vectors/background">Background vector created by starline - www.freepik.com</a>
