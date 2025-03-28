# Docker Compose Setup for PostgreSQL and External Node

This repository contains a Docker Compose setup for running a PostgreSQL database and an external node service. The setup is designed to be used with a specific external node version, and it configures the PostgreSQL instance with custom settings tailored for performance.

The Zksync External Node [Docs](https://github.com/matter-labs/zksync-era/blob/main/docs/src/guides/external-node/00_quick_start.md) is a great source of information if deeper node customization is desired.

## Prerequisites

- Docker
- Docker Compose

## Usage

1. Clone this repository.
2. Ensure Docker and Docker Compose are installed.
3. For `treasureTopaz`:
   1. Run `docker-compose --file ./docker-compose/treasuretopaz-external-node.yml up -d` to start the services.
   2. The services will be up and running as per the configurations defined in the `treasuretopaz-external-node.yml` for the testnet.
4. For `treasure`:
   1. Run `docker-compose --file ./docker-compose/treasure-external-node.yml up -d` to start the services.
   2. The services will be up and running as per the configurations defined in the `treasure-external-node.yml` for the testnet.

## Services

### 1. PostgreSQL

- **Image:** `postgres:14`
- **Exposed Ports:** `5430`
- **Volumes:** `mainnet-postgres` (Persistent storage for PostgreSQL data; for Treasure mainnet)
- **Environment Variables:**
  - `POSTGRES_PASSWORD`: Password for the PostgreSQL user.
  - `PGPORT`: Port number on which PostgreSQL will listen.
- **Custom Configuration Parameters:**
  - `max_connections=200`
  - `log_error_verbosity=terse`
  - `shared_buffers=2GB`
  - `effective_cache_size=4GB`
  - `maintenance_work_mem=1GB`
  - `checkpoint_completion_target=0.9`
  - `random_page_cost=1.1`
  - `effective_io_concurrency=200`
  - `min_wal_size=4GB`
  - `max_wal_size=16GB`
  - `max_worker_processes=16`
  - `checkpoint_timeout=1800`
- **Healthcheck:**
  - Runs a query to check if the PostgreSQL instance is ready.

### 2. External Node

- **Image:** `matterlabs/external-node:2.0-v27.1.1`
- **Depends on:** PostgreSQL service (ensures the service starts after PostgreSQL is healthy)
- **Ports:**
  - `3054` (Consensus public port)
  - `3060` (HTTP)
  - `3061` (WebSocket)
  - `3081` (Healthcheck)
  - `5000` (Additional port)
  - `3322` (Prometheus)
- **Volumes:** 
  - `mainnet-rocksdb` (Persistent storage for RocksDB data; for Treasure mainnet)
  - `./configs:/configs` (Configuration files)
- **Environment Variables:**
  - `DATABASE_URL`: Connection URL for the PostgreSQL instance.
  - `DATABASE_POOL_SIZE`: Size of the database connection pool.
  - `EN_HTTP_PORT`: Port for the HTTP server.
  - `EN_WS_PORT`: Port for the WebSocket server.
  - `EN_HEALTHCHECK_PORT`: Port for the health check.
  - `EN_PROMETHEUS_PORT`: Port for Prometheus metrics.
  - `EN_ETH_CLIENT_URL`: URL of the Ethereum client.
  - `EN_MAIN_NODE_URL`: URL of the main node.
  - `EN_L1_CHAIN_ID`: Layer 1 chain ID.
  - `EN_L2_CHAIN_ID`: Layer 2 chain ID.
  - `EN_L1_BATCH_COMMIT_DATA_GENERATOR_MODE`: Mode for batch commit data generator.
  - `EN_PRUNING_ENABLED`: Enable or disable pruning.
  - `EN_STATE_CACHE_PATH`: Path for the state cache.
  - `EN_MERKLE_TREE_PATH`: Path for the Merkle tree.
  - `EN_SNAPSHOTS_RECOVERY_ENABLED`: Enable or disable snapshots recovery.
  - `EN_SNAPSHOTS_OBJECT_STORE_BUCKET_BASE_URL`: Base URL for snapshots object store.
  - `EN_SNAPSHOTS_OBJECT_STORE_MODE`: Mode for snapshots object store.
  - `RUST_LOG`: Log level settings.
  - `EN_CONSENSUS_CONFIG_PATH`: Path to consensus configuration.
  - `EN_CONSENSUS_SECRETS_PATH`: Path to consensus secrets.

### 3. Grafana

- **Image:** `grafana/grafana:9.3.6`
- **Ports:** `3000` (Web interface)
- **Volumes:**
  - `mainnet-grafana-data` (Persistent storage for Grafana data; for Treasure mainnet)
  - `./grafana/provisioning` (Grafana provisioning configurations)
- **Environment Variables:**
  - `GF_AUTH_ANONYMOUS_ORG_ROLE`: Set to "Admin" for anonymous access
  - `GF_AUTH_ANONYMOUS_ENABLED`: Enable anonymous access
  - `GF_AUTH_DISABLE_LOGIN_FORM`: Disable login form

### 4. Prometheus

- **Image:** `prom/prometheus:v2.35.0`
- **Exposed Port:** `9090`
- **Volumes:**
  - `mainnet-prometheus-data` (Persistent storage for Prometheus data; for Treasure mainnet)
  - `./prometheus/prometheus.yml` (Prometheus configuration file)

## Volumes

### Treasure (Mainnet)
- `mainnet-postgres`: Stores PostgreSQL data.
- `mainnet-rocksdb`: Stores RocksDB data for the external node.
- `mainnet-prometheus-data`: Stores Prometheus metrics data.
- `mainnet-grafana-data`: Stores Grafana dashboards and configurations.

### Treasure Topaz (Testnet)
- `testnet-postgres`: Stores PostgreSQL data.
- `testnet-rocksdb`: Stores RocksDB data for the external node.
- `testnet-prometheus-data`: Stores Prometheus metrics data.
- `testnet-grafana-data`: Stores Grafana dashboards and configurations.
