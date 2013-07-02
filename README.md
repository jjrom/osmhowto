OSM Howto
=========
OpenStreetMap stuff

Store OSM data within elasticsearch
===================================

Install on Linux Ubuntu 
-----------------------

		# Install osmosis
		apt-get install osmosis

		# Install elasticsearch
		apt-get install openjdk-7-jre -y
		wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-0.90.2.tar.gz -O elasticsearch.tar.gz
		tar -xf elasticsearch.tar.gz
		rm elasticsearch.tar.gz
		mv elasticsearch-* elasticsearch
		mv elasticsearch /usr/local/share
		curl -L http://github.com/elasticsearch/elasticsearch-servicewrapper/tarball/master | tar -xz
		mv *servicewrapper*/service /usr/local/share/elasticsearch/bin/
		rm -Rf *servicewrapper*
		/usr/local/share/elasticsearch/bin/service/elasticsearch install
		ln -s `readlink -f /usr/local/share/elasticsearch/bin/service/elasticsearch` /usr/local/bin/rcelasticsearch

		# Install elasticsearch-osmosis plugin
		wget http://downloads.sourceforge.net/project/es-osmosis/releases/elasticsearch-osmosis-plugin-1.3.0.jar -O /usr/share/osmosis/elasticsearch-osmosis-plugin-1.3.0.jar 
		echo "org.openstreetmap.osmosis.plugin.elasticsearch.ElasticSearchWriterPluginLoader" > /etc/osmosis/osmosis-plugins.conf
		chmod 664 osmosis-plugins.conf 

		# Launch elasticsearch
		service elasticsearch start


Install on MacOS X
------------------

We suppose that brew is up and running (see http://mxcl.github.io/homebrew/)

		# Install osmosis
		brew install osmosis

		# Install elasticsearch
		brew install elasticsearch

		# Install elasticsearch-osmosis plugin
		wget http://downloads.sourceforge.net/project/es-osmosis/releases/elasticsearch-osmosis-plugin-1.3.0.jar -O /usr/local/opt/osmosis/libexec/lib/default/elasticsearch-osmosis-plugin-1.3.0.jar 
		chmod 664 /usr/local/opt/osmosis/libexec/lib/default/elasticsearch-osmosis-plugin-1.3.0.jar 
		sudo chgrp wheel /usr/local/opt/osmosis/libexec/lib/default/elasticsearch-osmosis-plugin-1.3.0.jar
		echo "org.openstreetmap.osmosis.plugin.elasticsearch.ElasticSearchWriterPluginLoader" > /usr/local/opt/osmosis/libexec/config/osmosis-plugins.conf
		chmod 664 osmosis-plugins.conf 

		# Install discovery plugin
		/usr/local/bin/plugin -install ncolomer/discovery

		# Launch elasticsearch
		ln -sfv /usr/local/opt/elasticsearch/*.plist ~/Library/LaunchAgents
		launchctl load ~/Library/LaunchAgents/homebrew.mxcl.elasticsearch.plist


Retrieve full OSM data
----------------------

		# Set OSM home directory
		export OSM_HOME=/home/data/osm

		# Download planet OSM
        mkdir -p $OSM_HOME/extract
        mkdir -p $OSM_HOME/planet
        mkdir -p $OSM_HOME/output
        wget -P $OSM_HOME/planet http://ftp.osuosl.org/pub/openstreetmap/pbf/planet-latest.osm.pbf

       	# Index data within elasticsearch
       	# Veeeery long so use nohup
		nohup osmosis --read-pbf $OSM_HOME/planet/planet-latest.osm.pbf --write-elasticsearch cluster.hosts="localhost" 2>nohup_err.txt 1>/dev/null &


Test installation
-----------------

		curl -XPOST http://localhost:9200/osm/_search?pretty=true -d '
		{
		  "query": {
		    "filtered": {
		      "query": {"match_all": {}},
		      "filter": {
		        "and": [
		          {
		            "term": {"tags.cuisine": "chinese"}
		          },
		          {
		            "geo_shape": {
		              "shape": {
		                "shape": {
		                  "type": "envelope",
		                  "coordinates": [[-52.3467,4.9588],[-52.2744,4.8944]]
		                }
		              }
		            }
		          }
		        ]
		      }
		    }
		  }
		}'



    
