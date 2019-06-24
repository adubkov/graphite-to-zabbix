# Graphite to Zabbix proxy
This tool allow handle alerts based on graphite metrics. It works as a proxy between graphite and zabbix. It use graphite as data source and zabbix as an alerting system.

Basic idea is schedule cronjob to run script, that makes request to zabbix server, gets filtered list of monitored metrics, makes appropriate requests to graphite, and sends metric back to zabbix.

## How to install
Easiest way to install g2zproxy over pip:
```bash
pip install graphite-to-zabbix
```

## How to use
To run g2zproxy once, just run cmd:
```bash
g2zproxy -z https://zabbix.local -zu {zabbixUser} -zp {zabbixPass} -g http://graphite.local
```
Note that g2zproxy will work with zabbix web api specified in `-z` argument, but it will send metrics to service specified in `/etc/zabbix/zabbix_agentd.conf`.


### Create zabbix metrics
First you need create few metrics to monitor in zabbix. I suppose you familar with zabbix template system. So, I just show to how to make one Item.

Suppose you want have an zabbix alert for some data from graphite. G2ZProxy will join `hostname` and graphite[`key`] from zabbix, and match it with graphite key.
An example graphite key `p-mem001.memcached.memcached_items-current.value` will be match to zabbix
```
host: p-mem001
key: graphite[memcached.memcached_items-current.value]
```

#### Graphite request with functions can be written in that manner.
Graphite request:
```
summarize(sum(statsd.drive_*error*),"5min","avg",true)
```
appropriate zabbix key:
```
graphite[statsd.derive_*error*; summarize(sum({metric}),\"5min\",\"avg\",true)]"
```

![alt tag](https://cloud.githubusercontent.com/assets/4600857/6422051/3749f056-be89-11e4-9dcd-78a1823f9e2a.png)

After you choosed which metric you want monitor, create zabbix item:

1. Create template:

![alt tag](https://cloud.githubusercontent.com/assets/4600857/6421550/e44f0b7e-be84-11e4-99bd-d9775e43b9a0.png)

2. Create item:

![alt tag](https://cloud.githubusercontent.com/assets/4600857/6421551/e44f33f6-be84-11e4-8cfc-175c6d052254.png)

3. Assign template to host:

![alt tag](https://cloud.githubusercontent.com/assets/4600857/6421552/e4510bea-be84-11e4-9b12-73b40eddfe34.png)

### Schedule cronjob task
Make cron task to run g2zproxy each minute:
```bash
$ crontab -e
```
```bash
# graphite to zabbix proxy
*/1 * * * * g2zproxy -z https://zabbix.local -zu {zabbixUser} -zp {zabbixPass} -g http://graphite.local > /dev/null 2>&1
```

### Performance
If g2zproxy seems to work slow, most likely you graphite is bottleneck. It's recomend to user distributed graphite cluster with loadbalanced to make it faster.

By default g2zproxy use 50th thread pool, you can change with `-t 10` argument.
