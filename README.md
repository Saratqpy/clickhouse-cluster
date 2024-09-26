# ClickHouse Cluster with ClickHouse Keeper

This project sets up a **ClickHouse Cluster** with one shard and two replicas using **ClickHouse Keeper**. Each component is containerized using Docker Compose.

## Features
- **1 Shard, 2 Replicas**: Provides data replication and failover capabilities.
- **ClickHouse Keeper**: Manages cluster state and leader election.
- **Separation of Components**: Each ClickHouse and Keeper instance runs in separate Docker Compose files, allowing for flexibility in deployment across different servers.

## Project Structure
- `clickhouse-01`: First replica of the ClickHouse database.
- `clickhouse-02`: Second replica of the ClickHouse database.
- `clickhouse-keeper-01`, `clickhouse-keeper-02`, `clickhouse-keeper-03`: ClickHouse Keeper instances.
- `ch-proxy`: Proxy to route requests to the ClickHouse cluster.

## Setup

1. Clone the repository:
    ```bash
    git clone https://github.com/saratqpy/clickhouse-cluster.git
    ```

2. Start the containers using Docker Compose:
    ```bash
    docker-compose up -d
    ```

3. Verify the setup by connecting to the cluster:
    ```bash
    docker exec -it clickhouse-01 clickhouse-client
    ```

## Creating the Clustered Database and Tables

1. Open DBeaver or any SQL client and create a new SQL script:
    ```sql
    CREATE DATABASE test ON CLUSTER cluster_1S_2R;
    ```

   The cluster name `cluster_1S_2R` is defined in the `config.xml` under the `remote_servers` section.

2. Create a replicated table across the cluster:
    ```sql
    CREATE TABLE test.my_table ON CLUSTER cluster_1S_2R (
        id UInt32,
        name String
    ) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/my_table', '{replica}') 
    ORDER BY id;
    ```

    This uses the `ReplicatedMergeTree` engine to maintain a copy of the data across replicas. The table is named `my_table`.

3. Create a distributed table:
    ```sql
    CREATE TABLE test.my_table_distributed ON CLUSTER cluster_1S_2R AS test.my_table
    ENGINE = Distributed(cluster_1S_2R, test, my_table, rand());
    ```

    This will distribute data across shards, ensuring that if multiple shards exist, tables in the same shard hold the same data.

4. View cluster information:
    ```sql
    SELECT * FROM system.clusters;
    ```

## Inserting Data

There are two ways to insert data into the tables:

1. **Manual SQL Execution**:
    ```sql
    INSERT INTO test.my_table_distributed VALUES (1, 'abc');
    ```

   **Note**: Direct inserts will throw an error if one of the nodes is down.

2. **Using CH Proxy**:
   You can use CH Proxy to automatically detect and avoid sending requests to nodes that are down. Here's an example of inserting data through the proxy:
    ```bash
    curl -u admin_1S_2R:1 -X POST http://127.0.0.1:80/ -d "INSERT INTO test.my_table VALUES (1, 'abc')"
    ```

   - `admin_1S_2R`: Username set in the `config.yml` file for CH Proxy.
   - `1`: Password defined in the same file.
   - Port `80`: The configured port for CH Proxy.

## Querying Data

To query data and see which node executed the query:
```bash
curl -u admin_1S_2R:1 -X POST http://127.0.0.1:80/ -d "SELECT hostName(), * FROM test.my_table LIMIT 1"
‍‍‍
```
The `hostName()` function returns the name of the node that executed the query.

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.

## Contributing
Feel free to open issues or submit pull requests for any improvements or new features.

## Contact
For any questions or feedback, please reach out via [email](saratqp7@gmail.com).
