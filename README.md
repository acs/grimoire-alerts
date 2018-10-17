# grimoirelab-alerts
A framework for GrimoireLab Alerts based on [ElastAlert](https://github.com/Yelp/elastalert).



## Install

The first step is to create a Python venv with ElastAlert:

```
acs@~/devel/elastalert $ sudo pip2 install virtualenv
acs@~/devel/elastalert $ virtualenv -p /usr/bin/python2.7 venv
acs@~/devel/elastalert $ source venv/bin/activate
acs@~/devel/elastalert (venv) $ pip install elastalert
```

## Configuration

The global configuration for ElastAlert is stored in a file called [config.yaml](config.yaml).
It includes among other things the connection to ElasticSearch with the metadata. 
There is a different Elasticsearch per deployment with the data to track with alerts.

You need to adapt it to your own deployment [following the doc](https://elastalert.readthedocs.io/en/latest/running_elastalert.html#downloading-and-configuring)

## Creating rules for the alerts

The alerts are defined [using a rule file](https://elastalert.readthedocs.io/en/latest/running_elastalert.html#creating-a-rule).

grimoire-alerts already includes a library of [precreated template rules files](rules-available) that
are the ones that should be used. Adding new template rules is easy and can be proposed with pull requests.

Right now only [frequencies alerts exist](https://elastalert.readthedocs.io/en/latest/ruletypes.html#frequency). In them, the key is to define the index from
which to get the data, and the [filter](https://github.com/acs/grimoirelab-alerts/blob/master/rules-available/stackexchange-no-answers.yaml#L40) to track the items (for example the question without answers).

### Design of reusable rules (alerts)

In order to reuse the templates rules in different deployments, at the top of the rule file is included:

```
# Include the connection data

import: "elasticsearch-connection"
```

You need to create this `elasticsearch-connection` in each deployment with the correct Elasticsearch connection data for this deployment.

To deploy a rule for a specific environment you just need to copy the template rule file
and change the name of the rule. Rule names must be unique inside a ElastAlert execution.

The names of the template rules files follow the schema: `datasource-name_of_metric.yaml`. For example `git-commits.yaml`.

## Creating the alerts for a deployment

For creating a new deployment with the commits alert active:

```
acs@~/devel/grimoirelab-alerts (venv) $ mkdir -p rules-enabled/bitergia.biterg.io/
acs@~/devel/grimoirelab-alerts (venv) $ vi rules-enabled/bitergia.biterg.io/elasticsearch-connection
acs@~/devel/grimoirelab-alerts (venv) $ cp rules-available/git-commits.yaml rules-enabled/bitergia.biterg.io/
acs@~/devel/grimoirelab-alerts (venv) $ vi rules-enabled/bitergia.biterg.io/git-commits.yaml
```

To create a second one with alerts for tracking answers in stackexchange:

```
acs@~/devel/grimoirelab-alerts (venv) $ mkdir -p rules-enabled/openshiftio.biterg.io/
acs@~/devel/grimoirelab-alerts (venv) $ vi rules-enabled/openshiftio.biterg.io/elasticsearch-connection
acs@~/devel/grimoirelab-alerts (venv) $ cp rules-available/stackexchange-no-answers.yaml rules-enabled/openshiftio.biterg.io/
acs@~/devel/grimoirelab-alerts (venv) $ vi  rules-enabled/openshiftio.biterg.io/stackexchange-no-answers.yaml
```

Remember than you must change the rule name so it is unique across all ElastAlert execution.

## Creating the alerts for a deployment based on mordred

An script called `m2a.py` will do automatically the deployment in mordred based
configuration. Using the mordred config file, all the data sources active will be
found and all the alerts for them will be created automatically.


## Testing the rules

The rules included in the library are already tested, but you need to define
the connection data in the file `elasticsearch-connection`. Once you have done it
you can check that the connection and the alert is working with:

```
acs@~/devel/grimoirelab-alerts (venv) $ elastalert-test-rule rules-enabled/bitergia.biterg.io/git-commits.yaml
....
Would have written the following documents to writeback index (default is elastalert_status):

elastalert_status - {'hits': 11, 'matches': 0, ...
```

## Running ElastAlert

ElastAlert will track the alerts in all the deployments configured.

```
acs@~/devel/grimoirelab-alerts (venv) $ ~/devel/elastalert/venv/bin/python -m elastalert.elastalert --verbose
INFO:elastalert:Starting up
INFO:elastalert:Queried rule Commits (Example frequency rule) from 2018-10-15 19:29 CEST to 2018-10-15 23:29 CEST: 3 / 3 hits
INFO:elastalert:Queried rule Commits (Example frequency rule) from 2018-10-15 23:29 CEST to 2018-10-16 03:29 CEST: 0 / 0 hits
INFO:elastalert:Queried rule Commits (Example frequency rule) from 2018-10-16 03:29 CEST to 2018-10-16 07:29 CEST: 3 / 3 hits
INFO:elastalert:Queried rule Commits (Example frequency rule) from 2018-10-16 07:29 CEST to 2018-10-16 11:29 CEST: 3 / 3 hits
INFO:elastalert:Queried rule Commits (Example frequency rule) from 2018-10-16 11:29 CEST to 2018-10-16 15:29 CEST: 0 / 0 hits
INFO:elastalert:Queried rule Commits (Example frequency rule) from 2018-10-16 15:29 CEST to 2018-10-16 18:47 CEST: 2 / 2 hits
INFO:elastalert:Ran Commits (Example frequency rule) from 2018-10-15 19:29 CEST to 2018-10-16 18:47 CEST: 11 query hits (0 already seen), 0 matches, 0 alerts sent
INFO:elastalert:Sleeping for 56.614832 seconds
^CINFO:elastalert:SIGINT received, stopping ElastAlert...
```