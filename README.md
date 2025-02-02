# shumway

[![Build Status](https://travis-ci.org/spotify/shumway.svg?branch=master)](https://travis-ci.org/spotify/shumway) [![Test Coverage](https://codecov.io/github/spotify/shumway/branch/master/graph/badge.svg)](https://codecov.io/github/spotify/shumway)

A micro library for sending metrics to a [FFWD](https://github.com/spotify/ffwd) agent.

## Requirements

* Either Python 2.7 or 3.6. Tests pass on Python 3.4, 3.5, 3.7, and PyPy 5.8 (2.7.13).
* Support for Linux & OS X

## To Use

```sh
(env) $ pip install shumway
```

### Counters

Create a default counter and send to FFWD:

```python
import shumway

mr = shumway.MetricRelay(SERVICE_NAME)
mr.incr(METRIC_NAME)
mr.flush()
```

#### Initialize a counter with a value

```python
import shumway

mr = shumway.MetricRelay(SERVICE_NAME)
counter = shumway.Counter(metric_name, SERVICE_NAME, value=10)
mr.set_counter(metric_name, counter)
mr.incr(metric_name)
mr.flush()
```

#### Different increment values

Create a named counter and increment by a value different than 1:

```python
import shumway

mr = shumway.MetricRelay(SERVICE_NAME)
mr.incr(METRIC_NAME, 2)
mr.flush()
```

#### Custom Counter Attributes

Set custom attributes for metrics:

```python
import shumway

mr = shumway.MetricRelay(SERVICE_NAME)
counter = shumway.counter(metric_name, SERVICE_NAME,
                          {attr_1: value_1,
                           attr_2: value_2})

mr.set_counter(metric_name, counter)
mr.incr(metric_name)
mr.flush()

```

**NB:** If you use duplicate names when calling `set_counter` it will overwrite the
counter. You will likely want to use a unique metric name for each set of
attributes you are setting.


### Timers

```python
import shumway

mr = shumway.MetricRelay(SERVICE_NAME)
timer = mr.timer('timing-this-thing')

with timer:
    ...task you want to time

mr.flush()
```

### Custom Timer Attributes
Timers can also be created independently in order to set custom attributes:

```python
import shumway

mr = shumway.MetricRelay(SERVICE_NAME)
timer = shumway.Timer('timing-this-thing', SERVICE_NAME,
                      {'attr_1': value_1, 'attr_2': value_2})

with timer:
    # ...task you want to time

mr.set_timer('timing-this-thing', timer)
mr.flush()
```

### Default attributes for non-custom metrics
MetricRelay can create metrics with a common set of attributes as well:

```python
import shumway

attributes = dict(foo='bar')
mr = shumway.MetricRelay(SERVICE_NAME, default_attributes=attributes)
```

### Sending Metrics

There are two ways to send metrics to the `ffwd` agent:

#### Emit one metric

You can emit a one-off, event-type metric immediately:

```python
import shumway

mr = shumway.MetricRelay('my-service')

# some event happened
mr.emit('a-successful-event', 1)

# some event happened with attributes
mr.emit('a-successful-event', 1, {'attr_1': value_1, 'attr_2': value_2})

# an event with a multiple value happened
mr.emit('a-successful-event', 5)
```

#### Flushing all metrics

For batch-like metrics, you can flush metrics once you're ready:

```python
import shumway

mr = shumway.MetricRelay('my-service')

# measure all the things
# time all the things

if not dry_run:
    mr.flush()
```


### Existing Metrics
Check for existence of metrics in the MetricRelay with `in`:

```pycon
>>> import shumway
>>> mr = shumway.MetricRelay('my-service')
>>> counter = shumway.Counter('thing-to-count', 'my-service', value=5)
>>> mr.set_counter('thing-to-count', counter)
>>> 'thing-to-count' in mr
True
>>> 'not-a-counter' in mr
False
```

### Custom FFWD agents

By default, `shumway` will send metrics to a local [`ffwd`](https://github.com/spotify/ffwd) agent at `127.0.0.1:19000`.

If your `ffwd` agent is elsewhere, then pass that information through when initializing the `MetricRelay`:

```python
import shumway

mr = shumway.MetricRelay(SERVICE_NAME, ffwd_ip='10.99.0.1', ffwd_port=19001)

# do the thing
```

# Developer Setup

For development and running tests, your system must have all supported versions of Python installed. We suggest using [pyenv](https://github.com/yyuu/pyenv).

## Setup

```sh
$ git clone git@github.com:spotify/shumway.git && cd shumway
# make a virtualenv
(env) $ pip install -r dev-requirements.txt
```

## Running tests

To run the entire test suite:

```sh
# outside of the virtualenv
# if tox is not yet installed
$ pip install tox
$ tox
```

If you want to run the test suite for a specific version of Python:

```sh
# outside of the virtualenv
$ tox -e py27
$ tox -e py36
```

To run an individual test, call `nosetests` directly:

```sh
# inside virtualenv
(env) $ nosetests test/metrics_test.py
```

# Code of Conduct

This project adheres to the [Open Code of Conduct][code-of-conduct]. By participating, you are expected to honor this code.

[code-of-conduct]: https://github.com/spotify/code-of-conduct/blob/master/code-of-conduct.md
