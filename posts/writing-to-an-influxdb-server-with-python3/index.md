# Writing to an InfluxDB server with Python3

> Write time-series data to InfluxDB from Python 3 for Grafana visualisation. Store sensor readings with nanosecond timestamps using the influxdb-client library.

Source: https://exitcode0.net/posts/writing-to-an-influxdb-server-with-python3/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-01-19
Updated: 2026-10-08
Tags: grafana, influxdb, python



A quick GitHub gist for anyone looking to write to an InfluxDB server with Python3. This is a generic function that accepts three inputs; as an example, I am using temperature data in degrees Celsius.

*Turning DHT11 readings into beautiful graphs in Grafana*
Once you have your data in influxDB, a great way to visualise it is using [Grafana](https://grafana.com/ "https://grafana.com/"). I hope to bring you more posts in the future about visualising your python data with Grafana.

[https://gist.github.com/Tombo1001/1f096ca224f0e61ac66548e2abd36a25](https://gist.github.com/Tombo1001/1f096ca224f0e61ac66548e2abd36a25 "https://gist.github.com/Tombo1001/1f096ca224f0e61ac66548e2abd36a25")

Now that you have the basics of writing events to influxDB with Python, feel free to leave links in the comments on the wonderful things you were able to graph with Grafana.

---

More Python Guides
------------------

* [CALCULATING COMPOUND INTEREST WITH PYTHON 3](https://exitcode0.net/posts/calculating-compound-interest-with-python-3/)
* [HOW TO CALL A BASH COMMAND WITH VARIABLES IN PYTHON](https://exitcode0.net/posts/how-to-call-a-bash-command-with-variables-in-python/ "https://exitcode0.net/posts/how-to-call-a-bash-command-with-variables-in-python/")
* [MONITOR YOUR PUBLIC IP ADDRESS WITH PYTHON](https://exitcode0.net/posts/monitor-your-public-ip-address-with-python/ "https://exitcode0.net/posts/monitor-your-public-ip-address-with-python/")

Worth noting that all of the above data could find its way onto a Grafana dashboard… the possibilities are endless!



---
Markdown version of https://exitcode0.net/posts/writing-to-an-influxdb-server-with-python3/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
