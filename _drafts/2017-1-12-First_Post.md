---
layout: post
title: Observations on MTA turnstile data (part 1)
published: true
---

The first week at Metis' Data Science Bootcamp, we're assigned to teams and tasked with using MTA data to help a fictional client place street teams at NYC subway stations. What interested me most about this project were the quirks and anomalies I observed in the MTA data. Though the assigment was fictional, the data is real, and these observations might help other Data Science students/practioners make sense of it.

### Where are they?

A Google search of `MTA turnsile` will tell you the MTA turnstile data is hosted at http://web.mta.info/developers/turnstile.html. There you'll find files containing the cumulative entry/exit counts for every turnstile in the NYC subway system.

Each file, named `turnstile_YYMMDD.txt`, contains seven days of data from Saturday thru Friday of the preceeding week (e.g., the file named `160903` contains the data for 2016-08-27 through 2017-09-02). The MTA website describes the file layout so I won't go into those details here.

### Downloading the files

The files are about ~15MB each and take while to download, so one of the first things I did was write Python functions to automatically identify, download, and cache the files. I could done this manually, but that sort of grunge work is what computers are for.

This code identifies the files to download:

```python
def get_turnstile_urls(count=-4, start=datetime.date.today()):
    """Identify MTA data files, given a start date and count.
    
    When count is positive, returns files after start date. When count
    is negative, returns files preceeding start date. Files are returned 
    in chronological order. When called with no arguments, returns the four
    most recent files.
    """

    MTA_URL_FMT = 'http://web.mta.info/developers/data/nyct/turnstile/turnstile_%y%m%d.txt'
    ONE_WEEK = datetime.timedelta(days=7)

    dt_first = start + datetime.timedelta(days=5 - start.weekday())
    if dt_first < start:
        dt_first += ONE_WEEK
        
    if count < 0:
        dt_first += count * ONE_WEEK
        count = abs(count)
        
    dates = [dt_first + ONE_WEEK * i for i in range(count)]
    return [datetime.date.strftime(d, MTA_URL_FMT) for d in dates]
```
And here's the code that uses the generated URLs to download and cache the files:

```python
def load_data(urls):
    """Import turnstile data into Pandas data frame, given list of URLs
    
    Downloads each file and saves a local copy. On subsequent calls,
    reads the cached local copy.
    """
    frames = []
    for url in urls:
        name = os.path.basename(url)
        if os.path.exists(name):
            print('reading local:', name)
            frames.append(pd.read_csv(name))
        else:
            print('reading:', url)
            frames.append(pd.read_csv(url))
            frames[-1].to_csv(name, index=False)
    return pd.concat(frames)
```
With these defined you can get the last four data files into a Pandas date frame with:

```python
df = load_data(get_turnstile_urls())
```
With those basics out of the way, my next post will describe the quirks you'll find in this data set.