# Redis High-Availability Cluster with Sentinel

A production-ready Redis cluster setup using Docker Compose with a master-slave replication configuration and Sentinel for automatic failover.

## Architecture

- **Redis Master**: Primary write node with persistence
- **Redis Slaves (2)**: Read replicas that sync with master
- **Redis Sentinel**: Monitors master/slave health and manages failover

## System Requirements

- Docker and Docker Compose
- Recommended: 4 CPU cores and 8GB RAM minimum

## Quick Start

1. Clone this repository
2. Edit `.env` file to set strong passwords and tune memory settings
3. Start the cluster:

```bash
docker-compose up -d
```

## Configuration

### Environment Variables

Edit the `.env` file to configure:

- `REDIS_PASSWORD`: Authentication password (use a strong one!)
- `SENTINEL_QUORUM`: Number of sentinels required to agree for failover (default: 2)
- `SENTINEL_DOWN_AFTER`: Time in ms to wait before considering master down (default: 5000)
- `SENTINEL_FAILOVER_TIMEOUT`: Time in ms allowed for failover (default: 10000)
- `REDIS_MAX_MEMORY_MASTER`: Memory limit for master node (default: 2gb)
- `REDIS_MAX_MEMORY_SLAVE`: Memory limit for slave nodes (default: 1gb)

### Advanced Configuration

For more advanced tuning, modify:

- `redis-master.conf` - Master node configuration
- `redis-slave.conf` - Slave node configuration
- `sentinel.conf` - Sentinel configuration

## Connecting to Redis

### Master (Write operations)

```bash
docker exec -it redis-master redis-cli -a $REDIS_PASSWORD
# OR
redis-cli -h localhost -p 6379 -a your_redis_password
```

### Slaves (Read operations)

```bash
docker exec -it redis-slave-1 redis-cli -a $REDIS_PASSWORD
# OR
redis-cli -h localhost -p 6380 -a your_redis_password
redis-cli -h localhost -p 6381 -a your_redis_password
```

### Sentinel (Monitoring and querying cluster state)

```bash
docker exec -it redis-sentinel redis-cli -p 26379
# OR
redis-cli -h localhost -p 26379
```

To find the current master:

```
sentinel get-master-addr-by-name mymaster
```

## Production Considerations

1. **Security**: 
   - Use a strong Redis password in `.env`
   - Consider adding SSL/TLS termination in front of Redis
   - Restrict access to Redis ports at the network level
   - Disable dangerous commands with `rename-command` in config files

2. **Performance**:
   - Tune `maxmemory` and `maxmemory-policy` based on your workload
   - Adjust `tcp-backlog` for high-concurrency workloads
   - Set `io-threads` for CPU-intensive operations
   - Enable `activedefrag` for better memory utilization

3. **Monitoring**:
   - Set up monitoring for Redis metrics (memory, connections, etc.)
   - Monitor Sentinel logs for failover events
   - Check replication status with `info replication` command

## Troubleshooting

Check container logs:

```bash
docker logs redis-master
docker logs redis-slave-1
docker logs redis-slave-2
docker logs redis-sentinel
```

Check replication status:

```bash
docker exec -it redis-master redis-cli -a $REDIS_PASSWORD info replication
```

Verify Sentinel configuration:

```bash
docker exec -it redis-sentinel redis-cli -p 26379 sentinel master mymaster
```

## License

This project is licensed under the MIT License - see the LICENSE file for