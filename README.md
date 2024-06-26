[![Build Status](https://travis-ci.com/o19s/elasticsearch-learning-to-rank.svg?branch=master)](https://travis-ci.com/o19s/elasticsearch-learning-to-rank)

The Elasticsearch Learning to Rank plugin uses machine learning to improve search relevance ranking. It's powering search at places like Wikimedia Foundation and Snagajob!

# What this plugin does...

This plugin:

- Allows you to store features (Elasticsearch query templates) in Elasticsearch
- Logs features scores (relevance scores) to create a training set for offline model development
- Stores linear, xgboost, or ranklib ranking models in Elasticsearch that use features you've stored
- Ranks search results using a stored model

## Where's the docs?

We recommend taking time to [read the docs](http://elasticsearch-learning-to-rank.readthedocs.io). There's quite a bit of detailed information about learning to rank basics and how this plugin can ease learning to rank development. 

You can also participate in regular [trainings](http://opensourceconnections.com/events/training) on Elasticsearch Learning to Rank, which support the free work done on this plugin.

## I want to jump in!

The demo lives in another repo now, [Hello LTR](https://github.com/o19s/hello-ltr) and it has both ES and Solr example. Follow the directions for Elasticsearch in the README to set up the environment and start with the [notebooks/elasticsearch/tmdb/hello-ltr.ipynb](https://github.com/o19s/hello-ltr/blob/master/notebooks/elasticsearch/tmdb/hello-ltr%20(ES).ipynb). Have fun!

# Installing

See the full list of [prebuilt versions](https://github.com/o19s/elasticsearch-learning-to-rank/releases) and select the version that matches your Elasticsearch version. If you don't see a version available, see the link below for building or file a request via [issues](https://github.com/o19s/elasticsearch-learning-to-rank/issues).

To install, you'd run a command like this but replacing with the appropriate prebuilt version zip:

`./bin/elasticsearch-plugin install https://github.com/o19s/elasticsearch-learning-to-rank/releases/download/v1.5.4-es7.11.2/ltr-plugin-v1.5.4-es7.11.2.zip`

(It's expected you'll confirm some security exceptions, you can pass `-b` to `elasticsearch-plugin` to automatically install)

If you already are running Elasticsearch, don't forget to restart!

# Known issues
As any other piece of software, this plugin is not exempt from issues. Please read the [known issues](KNOWN_ISSUES.md) to learn about the current issues that we are aware of. This file might include workarounds to mitigate them when possible.

# Build and Deploy Locally

Notes if you want to dig into the code or build for a version there's no build for, please feel free to run the build and installation process yourself:

```
./gradlew clean check
./bin/elasticsearch-plugin install file:///path/to/elasticsearch-learning-to-rank/build/distributions/ltr-<LTR-VER>-es<ES-VER>.zip
```

# How to Contribute

For more information on helping us out (we need your help!), developing with the plugin, creating docs, etc please read [CONTRIBUTING.md](/CONTRIBUTING.md).

## Elastic Release Support
We do our best to officially support `*.*.1` releases of Elasticsearch.  If you have a need for "dot-oh" compatibility or a version we don't support please consider submitting a PR.

 
# Who built this?
- [Initially developed](http://opensourceconnections.com/blog/2017/02/14/elasticsearch-learning-to-rank/) at [OpenSource Connections](http://opensourceconnections.com).
- Significant contributions by [Wikimedia Foundation](https://wikimediafoundation.org/wiki/Home), [Snagajob Engineering](https://engineering.snagajob.com/), [Bonsai](https://bonsai.io/), and [Yelp Engineering](https://engineeringblog.yelp.com/)
- Thanks to [Jettro Coenradie](https://amsterdam.luminis.eu/author/jettro/) for porting to ES 6.1

## Other Acknowledgments & Stuff To Read
- Bloomberg's [Learning to Rank work for Solr](https://issues.apache.org/jira/browse/SOLR-8542)
- Our Berlin Buzzwords Talk, [We built an Elasticsearch Learning to Rank plugin. Then came the hard part](https://2017.berlinbuzzwords.de/17/session/we-built-elasticsearch-learning-rank-plugin-then-came-hard-part.html)
- Blog article on [How is Search Different from Other Machine Learning Problems](http://opensourceconnections.com/blog/2017/08/03/search-as-machine-learning-prob/)
- Also check out our other relevance/search thingies: book [Relevant Search](http://manning.com/books/relevant-search), projects [Elyzer](http://github.com/o19s/elyzer), [Splainer](http://splainer.io), and [Quepid](http://quepid.com)
