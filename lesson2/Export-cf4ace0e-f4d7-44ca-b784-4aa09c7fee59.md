# TIDB-Benchmark实践手册

## 本地电脑参数配置

[配置列表](https://www.notion.so/3da795ab07e94b22b27a3caced167495)

---

# 通过TIUP部署本地单机版本的ti-server，ti-monitor，tidb，tikv

执行命令：tiup playground v4.0.0 --pd.config ./pd.toml

- pd.toml

    ```yaml
    # PD Configuration.

    name = "pd"
    data-dir = "default.pd"

    client-urls = "http://127.0.0.1:2379"
    ## if not set, use ${client-urls}
    advertise-client-urls = ""

    peer-urls = "http://127.0.0.1:2380"
    ## if not set, use ${peer-urls}
    advertise-peer-urls = ""

    initial-cluster = "pd=http://127.0.0.1:2380"
    initial-cluster-state = "new"

    ## set different tokens to prevent communication between PDs in different clusters.
    # initial-cluster-token = "pd-cluster"

    lease = 3
    tso-save-interval = "3s"

    enable-prevote = true

    [security]
    ## Path of file that contains list of trusted SSL CAs. if set, following four settings shouldn't be empty
    cacert-path = ""
    ## Path of file that contains X509 certificate in PEM format.
    cert-path = ""
    ## Path of file that contains X509 key in PEM format.
    key-path = ""

    cert-allowed-cn = ["example.com"]

    [log]
    level = "info"

    ## log format, one of json, text, console
    # format = "text"

    ## disable automatic timestamps in output
    # disable-timestamp = false

    # file logging
    [log.file]
    # filename = ""
    ## max log file size in MB
    # max-size = 300
    ## max log file keep days
    # max-days = 28
    ## maximum number of old log files to retain
    # max-backups = 7

    [metric]
    ## prometheus client push interval, set "0s" to disable prometheus.
    interval = "15s"
    ## prometheus pushgateway address, leaves it empty will disable prometheus.
    address = ""

    [pd-server]
    ## the metric storage is the cluster metric storage. This is use for query metric data.
    ## Currently we use prometheus as metric storage, we may use PD/TiKV as metric storage later.
    ## For usability, recommended to temporarily set it to the prometheus address, eg: http://127.0.0.1:9090
    metric-storage = ""

    [schedule]
    max-merge-region-size = 20
    max-merge-region-keys = 200000
    split-merge-interval = "1h"
    max-snapshot-count = 3
    max-pending-peer-count = 16
    max-store-down-time = "30m"
    leader-schedule-limit = 4
    region-schedule-limit = 2048
    replica-schedule-limit = 64
    merge-schedule-limit = 8
    hot-region-schedule-limit = 4
    ## There are some policies supported: ["count", "size"], default: "count"
    # leader-schedule-policy = "count"
    ## When the score difference between the leader or Region of the two stores is
    ## less than specified multiple times of the Region size, it is considered in balance by PD.
    ## If it equals 0.0, PD will automatically adjust it.
    # tolerant-size-ratio = 0.0

    ## This three parameters control the merge scheduler behavior.
    ## If it is true, it means a region can only be merged into the next region of it.
    # enable-one-way-merge = false
    ## If it is true, it means two region within different tables can be merged.
    ## This option only works when key type is "table".
    # enable-cross-table-merge = false

    ## customized schedulers, the format is as below
    ## if empty, it will use balance-leader, balance-region, hot-region as default
    # [[schedule.schedulers]]
    # type = "evict-leader"
    # args = ["1"]

    [replication]
    ## The number of replicas for each region.
    max-replicas = 3
    ## The label keys specified the location of a store.
    ## The placement priorities is implied by the order of label keys.
    ## For example, ["zone", "rack"] means that we should place replicas to
    ## different zones first, then to different racks if we don't have enough zones.
    location-labels = []
    ## Strictly checks if the label of TiKV is matched with location labels.
    # strictly-match-label = false

    [label-property]
    ## Do not assign region leaders to stores that have these tags.
    # [[label-property.reject-leader]]
    # key = "zone"
    # value = "cn1

    [dashboard]
    ## Configurations below are for the TiDB Dashboard embedded in the PD.

    ## The path of the CA certificate used to verify the TiDB server in TLS.
    # tidb-cacert-path = ""
    ## The path of the certificate used to connect to TiDB server in TLS.
    # tidb-cert-path = ""
    ## The path of the certificate private key.
    # tidb-key-path = ""

    ## The public path prefix to serve Dashboard urls. It can be set when Dashboard
    ## is running behind a reverse proxy. Do not configure it if you access
    ## Dashboard directly.
    # public-path-prefix = "/dashboard"

    ## When enabled, request will be proxied to the instance running Dashboard
    ## internally instead of result in a 307 redirection.
    # internal-proxy = false

    ## When enabled, usage data will be sent to PingCAP for improving user experience.
    # enable-telemetry = true
    ```

---

# 测试阶段

## Sysbench测试

[参数配置](https://www.notion.so/ca00273519a144ab9d6bb9585a24a8f1)

> Sysbench数据load

sysbench oltp_update_non_index --config-file=ticonfig --threads=4 --tables=4 --table_size=10000 prepare

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled.png)

- oltp_point_select

    sysbench oltp_point_select --config-file=ticonfig --threads=4 --tables=4 --table_size=10000 run

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%201.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%201.png)

    sysbench oltp_point_select --config-file=ticonfig --threads=10 --tables=4 --table_size=10000 run

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%202.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%202.png)

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%203.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%203.png)

    TIKV Cluster QPS

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%204.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%204.png)

    TIKV Grpc duration

## GO-YSCB测试

[参数配置](https://www.notion.so/2fd234a3ffdc406d80baede14da751bc)

### 负载测试

- workload-a

    > 通过go-ycsb来做数据加载

    ./bin/go-ycsb load mysql -P workloads/workloada -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 4

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%205.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%205.png)

    ./bin/go-ycsb load mysql -P workloads/workloada -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 1

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%206.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%206.png)

    > 通过go-yscb做test

    ./bin/go-ycsb run mysql -P workloads/workloada -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 4

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%207.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%207.png)

    ./bin/go-ycsb run mysql -P workloads/workloada -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 4

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%208.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%208.png)

- workload-b

    > 通过go-ycsb来做数据加载

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%209.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%209.png)

    > 通过go-yscb做test

    5thread, 10000record

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2010.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2010.png)

    1thread 10000records

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2011.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2011.png)

- workload-c

    > 通过go-ycsb来做数据加载

    load 1thread 10000records

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2012.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2012.png)

    load 4thread 10000records

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2013.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2013.png)

    > 通过go-yscb做test

    1thread 10000record 

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2014.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2014.png)

    4thread 10000record 

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2015.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2015.png)

- workload-d

    > 通过go-ycsb来做数据加载

    ./bin/go-ycsb load mysql -P workloads/workloadd -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 1

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2016.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2016.png)

    ./bin/go-ycsb load mysql -P workloads/workloadd -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 4

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2017.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2017.png)

    > 通过go-ycsb来做test

    ./bin/go-ycsb run mysql -P workloads/workloadd -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 1

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2018.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2018.png)

    ./bin/go-ycsb run mysql -P workloads/workloadd -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 4

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2019.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2019.png)

- workload-e

    > 通过go-ycsb来做数据加载

    ./bin/go-ycsb load mysql -P workloads/workloade -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 1

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2020.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2020.png)

    ./bin/go-ycsb load mysql -P workloads/workloade -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 4

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2021.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2021.png)

    > 通过go-ycsb来做test

    ./bin/go-ycsb run mysql -P workloads/workloade -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 1

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2022.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2022.png)

    ./bin/go-ycsb run mysql -P workloads/workloade -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 4

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2023.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2023.png)

- workload-f

    > 通过go-ycsb来做数据加载

    ./bin/go-ycsb load mysql -P workloads/workloadf -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 1

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2024.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2024.png)

    ./bin/go-ycsb load mysql -P workloads/workloadf -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 4

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2025.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2025.png)

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2026.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2026.png)

    > 通过go-ycsb来做test

    ./bin/go-ycsb run mysql -P workloads/workloade -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 1

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2027.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2027.png)

    ./bin/go-ycsb run mysql -P workloads/workloade -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 4

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2028.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2028.png)

    ![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2029.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2029.png)

---

## GO-TPC测试

[参数配置](https://www.notion.so/05b761334cf54b108a259f95cbb6e9c8)

## 数据准备

./bin/go-tpc tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 10 prepare

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2030.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2030.png)

## 数据查询

./bin/go-tpc tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 2 run

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2031.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2031.png)

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2032.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2032.png)

./bin/go-tpc tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 1 run

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2033.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2033.png)

./bin/go-tpc tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 1 run --threads 4

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2034.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2034.png)

---

# Grafana QPS

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2035.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2035.png)

## TIDB Duration & QPS

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2036.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2036.png)

Duration & QPS

## TIKV-Detail Cluster QPS & CPU

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2037.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2037.png)

QPS

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2038.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2038.png)

CPU

## TIKV-Detail Server QPS & CPU

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2039.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2039.png)

## TIKV-GRPC CPU & Duration

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2040.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2040.png)

CPU

![TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2041.png](TIDB-Benchmark%E5%AE%9E%E8%B7%B5%E6%89%8B%E5%86%8C%20048d5bb8bde14eb6a0048c9bd4355929/Untitled%2041.png)

Duration

---

# 个人愚见

由于机器限制，我只能通过tiup playground来测试tidb相关的性能，机器参数在上面，我个人的想法如下：

- 线程数跟机器逻辑CPU个数有关，超过CPU核数太多性能性价比弱了很多，可能是微增长甚至是负增长；
- 压测下来看，瓶颈不在CPU上面，会有若干次GC，导致整体时间飙升到1s，需要对参数做一些调优和配置；
- 一个Region Miss的时候，查询时间直接飙升到3.9s，速度下降了100倍。