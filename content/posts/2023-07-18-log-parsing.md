---
layout: post
date: 2023-07-18
title:  "Logs: Finding Structure in the Chaos"
ShowToc: true
TocSide: 'right'
TocOpen: true
draft: true
---

<!-- your comment text 


Do you wonder how modern tools analyze and gaub insights from logs?

The following repository contains a collections 


--------------------


Why?
Read logs faster

Nice dashboards

What?
This project online

Pandas

Plotly


How?
Find a dataset
Get the code
Go ham!


The Cray data

in

https://www.usenix.org/cfdr-data


--------------------
 -->

Did you ever get overwhelmed trying to understand all different log message of an application? In this post I'd like to share a solution to analyze application logs that combines different open source projects to produce an appealing dashboard summarizing all information.

We will use a github repository with a collection of algorithms to parse logs, [logparser](https://github.com/logpai/logparser), _plotly_ to create interactive plots, and _pandas_ to transform data.


For starters, we will by briefly explain the dependencies and datasets used, and later we show one example of a dashboard.


Let us get started!

# Dependencies

## Logparser

The first and main dependency is the repository [logparser](https://github.com/logpai/logparser) which consists of a collection of implementations of algorithms parse logs. Furthermore, the repository includes the corresponding articles explaining each algorithm to great extent.

One of the algorithm that works quite well is _Spell_ and we will use it throughout the post. _Spell_ works well in most scenarios since it is fast, requires no pre-defined log templates or regular expressions specific to the application, and parses logs in an online streaming fashion, so it is able to start parsing logs immediately.


The accuracy of _Spell_ is also supported by multiple benchmarks collected in [logparser](https://github.com/logpai/logparser) and its performance is often one of the best amongst the options tested.

This algorithm calculates the longest common sequences between log messages and groups them together depending on how big it is compared to the size of each log message. The hyperparameter _tau_ can be used to specify the cutoff of the ratio between the longest common sequence and the number of tokens of the log message in order to a log message to be grouped with other messages.

### Example




## Cray Data

We use log data shared by [usenix](https://www.usenix.org/cfdr-data), specifically the [log dataset](https://www.usenix.org/sites/default/files/4366-0809181018.tar.gz) corresponding to Cray Systems.


Here is a sample of the logs.

```text
[Tue Sep  9 20:18:34 2008]:

Boot Node Daemon starting at Tue Sep  9 20:18:34 2008.
[Tue Sep  9 20:18:34 2008]:Connecting to SMW '10.3.1.1'
[Tue Sep  9 20:18:34 2008]:done
[Tue Sep  9 20:18:34 2008]:topo class is RS_TOPO_CLASS_1
[Tue Sep  9 20:18:38 2008]:Event:Tue Sep  9 20:18:40 2008|ec_hsn_load_req Transfer a kernel image over HSN to a node|src:1b:3:s0||svc: svid=:9:s0:9:s0 nid=c0-0c0s0n3  bparams='' kpath='/tmp/boot/kernel.cpio.2.1.31HD-13426' flags=0x3
[Tue Sep  9 20:18:38 2008]:
***** HSN Booting 1 nodes at Tue Sep  9 20:18:38 2008
[Tue Sep  9 20:18:38 2008]:number of interfaces is 8
[Tue Sep  9 20:18:38 2008]:initialized portals event queue
[Tue Sep  9 20:18:41 2008]:loading index-file /tmp/boot_9lDixW/SNL0.load
[Tue Sep  9 20:18:41 2008]:Set comp mdh[0] to 0x0
[Tue Sep  9 20:18:41 2008]:Corresponding md is the following:
[Tue Sep  9 20:18:41 2008]:start  is 0x2b20e175b010
```


# Dashboard

Now we start the example.

Show the full picture immediatelly

## Parsing logs

The algorithm creates two files...

```python
import pandas as pd
from logparser.Spell import Spell  # use tau=0.8

t = "data/0809181018"
parser = Spell.LogParser(log_format="\[<Timestamp>\]:<Content>", indir=t, outdir=t, tau=0.5)
parser.parse("bnd.log")
```

Structured


| LineId | Timestamp                | Content                            | EventId  | EventTemplate                     | ParameterList       |
|--------|--------------------------|------------------------------------|----------|-----------------------------------|---------------------|
|     11 | Tue Sep  9 20:18:41 2008 | Corresponding md is the following: | 1cbf7853 | Corresponding md is the following | []                  |
|     12 | Tue Sep  9 20:18:41 2008 | start  is 0x2b20e175b010           | 365695dc | start is <*>                      | ['0x2b20e175b010']  |
|     13 | Tue Sep  9 20:18:41 2008 | length is 0x3ffb28                 | f3cb5b83 | length is <*>                     | ['0x3ffb28']        |
|     14 | Tue Sep  9 20:18:41 2008 | user_ptr is 0x3e9                  | 7d7147f1 | user_ptr is <*>                   | ['0x3e9']           |
|     15 | Tue Sep  9 20:18:41 2008 | Set comp mdh[1] to 0x1             | 2bab36e2 | Set comp <*> to <*>               | "['mdh[1]', '0x1']" |


## Timestamps

```python
(
    df
    .assign(
        Timestamp=lambda x0:
        x0["Timestamp"]
        .str.extract(r"\S+\s+(?P<m>\S+)\s+(?P<d>\S+)\s+(?P<t>\S+)\s+(?P<y>\S+)")
        .replace(dict(zip(month_abbr, range(13))))
        .assign(m=lambda x: x["m"].astype(str).str.rjust(2, "0"))
        .assign(d=lambda x: x["d"].astype(str).str.rjust(2, "0"))
        .pipe(lambda x: pd.to_datetime(x["y"] + "-" + x["m"] + "-" + x["d"] + " " + x["t"])))
)
```

## Plot Result

```python
import plotly.express as px

df = pd.read_csv(next(Path(t).glob("*_structured.csv")))

(
    px.histogram(
        df
        .assign(
            Timestamp=lambda x0:
            x0["Timestamp"]
            .str.extract(r"\S+\s+(?P<m>\S+)\s+(?P<d>\S+)\s+(?P<t>\S+)\s+(?P<y>\S+)")
            .replace(dict(zip(month_abbr, range(13))))
            .assign(m=lambda x: x["m"].astype(str).str.rjust(2, "0"))
            .assign(d=lambda x: x["d"].astype(str).str.rjust(2, "0"))
            .pipe(lambda x: pd.to_datetime(x["y"] + "-" + x["m"] + "-" + x["d"] + " " + x["t"])))
        [["Timestamp", "EventTemplate"]],
        x="Timestamp",
        color="EventTemplate",
        nbins=288)
    .update_layout(height=500, legend=dict(y=-0.5, orientation="h"), title="logs")
    .show()
)
```

![Plot Results](/images/blog/logs_histogram.png)






