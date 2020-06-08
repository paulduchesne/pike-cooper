# Pike-Cooper

This project works through the process of standing up a custom Wikibase instance via Docker and embedding film data. 

The test data is sourced from Andrew Pike and Ross Cooper's excellent overview of the early Australian film industry, *Australian Film 1900-1977*, which can be purchased here: https://www.roninfilms.com.au/video/2221/0/2245.html

Prerequisite steps:
1. Download and install Docker Desktop (https://www.docker.com/products/docker-desktop).
2. git clone wikibase-docker (https://github.com/wmde/wikibase-docker.git).
3. cd into wikibase-docker directory (make sure docker is running first) and run 'docker-compose up -d' to create container (this may take a while the first time as it needs to download all required components).
4. Once installed there should be a page visible at http://localhost:8181/wiki/Main_Page
5. Create an account, and duplicate those login details into the config.json file located at /notebooks/1_wikibase_instance/config.json

There are four notebooks included in the repo:

1. wikibase_instance: this notebook covers the injection of intial film data.
2. wikidata_extract: this is the extraction of primary film-related data from Wikidata for matching.
3. matching: the two datasets are compared and trusted matches identified.
4. linking: the discovered matches are embedded into the Wikibase instance.