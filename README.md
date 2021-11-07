## Presidentielle ELK
In this project we are concerned by the analysis of tweets about the _2022 French presidential election_. To do so we use ELK stack to retrieve, analyze and visualize tweets with the following hashtags : _#presidentielles2022, #presidentielle2022_. 

### Components and pipeline architecture

For convenience, we’ll use Docker and Docker Compose. Our pipeline is composed of four (4) docker containers, as follows : 

- **Logstash**: a service used to collect, process and forward events and log messages. In our case it's responsible of collecting tweets with the previous hashtags. An additional task will be to forward collected data into a backend database (MongoDB).
- **MongoDB**: the well-known NoSQL database used here to store collected tweets.
- **Elasticsearch**: a search engine providing a distributed capable full-text search.
- **Kibana**: a browser-based analytics platform used here as search interface for Elasticsearch and to provides visualization capabilities on top of the content indexed on Elasticsearch.

An overall overview of the pipeline is depicted in the figure bellow. More details on each component is provided next.

![Alt text](images/pipeline.png?raw=true "pipeline")

### Logstash

_Logstash_ is used to capture tweets and stream data into _MongoDB_ as a data store  and _ElasticSearch_ for indexing.  Retrieving twitter messages based on keyword is achieved by [twitter input plugin](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-twitter.html), defined in logstash configuration files, as follows. 

```
input {
  twitter {
    consumer_key => "<your_consumer_key>"
    consumer_secret => "<your_consumer_secret>"
    oauth_token => "your_oauth_token"
    oauth_token_secret => "your_oauth_token_secret"
    keywords => ["Presidentielle2022","Presidentielles2022"]
    full_tweet => true        
  }
}
```
The configuration file is located in  _pipeline_ directory, inside _logstash docker dir_.  The file is loaded automatically while building the docker image, using the following line (inside _Dockerfile_) : 
```
ADD pipeline/ /usr/share/logstash/pipeline/
```
Inside the same _Dockerfile_ file, note the following line :
```
RUN bin/logstash-plugin install --version=3.1.5 logstash-output-mongodb
```
In this line we install _Twitter logstash plugin_ (not installed by default in logstash). We explicitly precised the version 3.1.5, as the versions above seem to have some problems with the _MongoDB_ output.

>_For those who don’t already have a twitter developer account, you may get one by following the twitter [documentation](https://developer.twitter.com/en/docs/basics/getting-started)  [[1]](https://clementbm.github.io/elasticsearch/kibana/logstash/elk/sentiment%20analysis/2020/03/02/elk-sentiment-analysis-twitter-coronavirus.html).

### Output to Elasticsearch

_Logstash_ routes events to output plugins which can forward the events to a variety of external programs including _Elasticsearch_, local files and several message bus implementations [1].

In this project, we simply pass the output to _MongoDB_ as a data store and _Elacticsearch_ for indexation (and visualisation using _Kibana_) :

The final output is then :
```
  mongodb {
    isodate => true
    database => "twitter"
    collection => "doc"
    uri => "mongodb://mongodb:27017"
  }
  elasticsearch {
      hosts => ["elasticsearch:9200"]
      index => "presidentielle"
  }  
```

## Visualisation using Kibana

We can now access Kibana interface to get some insights. First we need to add the `presidentielle` index pattern by clicking on “Created index pattern”  (defined in the output). 

![index pattern](https://clementbm.github.io/assets/2020-03-02/index-pattern.png)

Then we can add some visualization chart by clicking on “Create visualization”



### What's next

- _Geographic data_ : Collected tweets may include fields with _geo_data_ information. To be visualized in maps, we need to do some transformation by defining a new _mapping_ to our index. Unfortunately, the percentage of users enabling such information is so small, especially in our context.
- _Sentiment Analysis_ : _Logstash_ and _Elaticsearch_ offer the possibility to include add-on plugins. In our case plugins related to _sentiment analysis_ may provide useful insights on users' perception regarding this presidential. Interested users are referred to the following articles for more details on how such tasks may be done in pipelines such ours, e.g. _logstash-filter-sentimentalizer_ [2], _openNLP_ [3], or those dedicated to french [4, 5, 6].
- _Security_ : For a step forward, a lot can be done to improve security. In fact, ELK stack allows the integration of a multitude of technologies affecting _authentification_ (LDAP, SSO, Kerberos, SAML, etc), _autorisation_ (user access rights and privileges for Elastiksearch and Kibana), _encryption_ (SSL/TLS), etc.

### Snapshots


![Alt text](images/discover.png?raw=true "discover")
![Alt text](images/view1.png?raw=true "view1")
![Alt text](images/view2.png?raw=true "view2")
![Alt text](images/view2.png?raw=true "view2")
![Alt text](images/search.png?raw=true "search")
![Alt text](images/mapping.png?raw=true "mapping")


## References

[1] [Sentiment analysis on twitter data with ELK](https://clementbm.github.io/elasticsearch/kibana/logstash/elk/sentiment%20analysis/2020/03/02/elk-sentiment-analysis-twitter-coronavirus.html)
[2] [logstash-filter-sentimentalizer logstash pluging](https://github.com/tylerjl/logstash-filter-sentimentalizer)
[3] [A DIY Twitter Tracking Tool Using the Elastic Stack](https://a2i2.deakin.edu.au/2020/11/05/a-diy-twitter-tracking-tool-using-the-elastic-stack/)
[4] [Language analyzers - French analyze](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html#french-analyzer)
[5] [Construire un bon analyzer français pour Elasticsearch](https://jolicode.com/blog/construire-un-bon-analyzer-francais-pour-elasticsearch#tldr)
[6] [French-phonetic-analyser plugin (token filter)](https://github.com/hcapitaine/french-phonetic-analyser)




