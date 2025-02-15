# Pike-Cooper

Dataset from Andrew Pike and Ross Cooper's [Australian Film 1900-1977](https://www.roninfilms.com.au/video/2221/0/2245.html) with [Wikidata](https://www.wikidata.org) identifiers.


### Queries

Note that these queries require the [pandas](https://pypi.org/project/pandas/) and [requests](https://pypi.org/project/requests/) libraries.

Is Wikidata ID valid? This query will return a dataframe of all entries which are no longer active, either due to redirection or deletion.

```python
import pandas
import pathlib
import requests

df = pandas.read_csv(pathlib.Path.cwd() / 'pike_cooper.csv')
wikidata_ids = ' '.join(['wd:'+x for x in df.wikidata.unique() if type(x) == str])

query = '''
    select ?wikidata 
    where {
        values ?wikidata {'''+wikidata_ids+'''}
        ?wikidata wdt:P31 ?type
    } '''

r = requests.get('https://query.wikidata.org/sparql', params={'format': 'json', 'query': query})
df2 = pandas.DataFrame(r.json()['results']['bindings']).dropna()
df2['wikidata'] = df2['wikidata'].apply(lambda x: x['value'].split('/')[-1])
df = df.loc[~df.wikidata.isin(df2.wikidata.unique())]

print(len(df))
df.head()
```

Federate from Wikidata, here pulling in "Country of Origin" (P495) data.

```python
import pandas
import pathlib
import requests

df = pandas.read_csv(pathlib.Path.cwd() / 'pike_cooper.csv')
wikidata_ids = ' '.join(['wd:'+x for x in df.wikidata.unique() if type(x) == str])

query = '''
    select ?wikidata ?countryLabel  
    where {
        values ?wikidata {'''+wikidata_ids+'''}
        ?wikidata wdt:P495 ?country .
        service wikibase:label { bd:serviceParam wikibase:language "en". }  
    } '''

r = requests.get('https://query.wikidata.org/sparql', params={'format': 'json', 'query': query})
df2 = pandas.DataFrame(r.json()['results']['bindings']).dropna()
df2['wikidata'] = df2['wikidata'].apply(lambda x: x['value'].split('/')[-1])
df2['countryLabel'] = df2['countryLabel'].apply(lambda x: x['value'])
df2 = df2.pivot_table(index=['wikidata'], aggfunc=lambda x: ', '.join(sorted(x.unique()))).reset_index()
df = pandas.merge(df, df2, on='wikidata', how='left')

print(len(df))
df.head()
```

Access third-party identifiers, here pulling in "IMDB ID" (P345).

```python
import pandas
import pathlib
import requests

df = pandas.read_csv(pathlib.Path.cwd() / 'pike_cooper.csv')
wikidata_ids = ' '.join(['wd:'+x for x in df.wikidata.unique() if type(x) == str])

query = '''
    select ?wikidata ?imdb  
    where {
        values ?wikidata {'''+wikidata_ids+'''}
        ?wikidata wdt:P345 ?imdb . 
    } '''

r = requests.get('https://query.wikidata.org/sparql', params={'format': 'json', 'query': query})
df2 = pandas.DataFrame(r.json()['results']['bindings']).dropna()
df2['wikidata'] = df2['wikidata'].apply(lambda x: x['value'].split('/')[-1])
df2['imdb'] = df2['imdb'].apply(lambda x: 'https://www.imdb.com/title/'+x['value'])
df = pandas.merge(df, df2, on='wikidata', how='left')

print(len(df))
df.head()
```