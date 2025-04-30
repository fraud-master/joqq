# 분산락(Distributed Lock)

분산락은 분산 시스템 환경에서 여러 서버나 프로세스가 공유 자원에 동시에 접근하는 것을 제어하기 위한 메커니즘입니다. 단일 서버 환경에서는 메모리 내 락이나 파일 락으로 충분하지만, 분산 환경에서는 모든 노드가 인식할 수 있는 특별한 형태의 락이 필요합니다.

## 분산락의 필요성

- **동시성 제어**: 여러 서버에서 동일한 자원에 접근할 때 데이터 일관성 유지
- **중복 작업 방지**: 배치 작업이나 스케줄러가 여러 서버에서 중복 실행되는 것을 방지
- **순차적 처리 보장**: 특정 작업이 순서대로 실행되어야 할 때 사용

## 주요 분산락 구현 방식

### 1. 데이터베이스 기반 분산락

```sql
-- MySQL을 이용한 분산락 예제
-- 락 획득 시도
INSERT INTO distributed_locks (lock_name, lock_owner, expiry_time)
VALUES ('resource_name', 'server_id', NOW() + INTERVAL 30 SECOND)
ON DUPLICATE KEY UPDATE 
    lock_owner = IF(expiry_time < NOW(), VALUES(lock_owner), lock_owner),
    expiry_time = IF(expiry_time < NOW(), VALUES(expiry_time), expiry_time);

-- 락 해제
DELETE FROM distributed_locks 
WHERE lock_name = 'resource_name' AND lock_owner = 'server_id';
```

### 2. Redis를 이용한 분산락

```java
// Redis를 이용한 분산락 구현 예제 (Java)
public class RedisDistributedLock {
    private JedisPool jedisPool;
    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";
    
    public boolean acquireLock(String lockKey, String requestId, int expireTime) {
        try (Jedis jedis = jedisPool.getResource()) {
            String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, 
                                    SET_WITH_EXPIRE_TIME, expireTime);
            return LOCK_SUCCESS.equals(result);
        }
    }
    
    public boolean releaseLock(String lockKey, String requestId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                        "return redis.call('del', KEYS[1]) else return 0 end";
        try (Jedis jedis = jedisPool.getResource()) {
            Object result = jedis.eval(script, Collections.singletonList(lockKey),
                                    Collections.singletonList(requestId));
            return Long.parseLong(result.toString()) == 1L;
        }
    }
}
```

### 3. ZooKeeper를 이용한 분산락

```java
// ZooKeeper를 이용한 분산락 구현 예제 (Java)
public class ZookeeperDistributedLock {
    private ZooKeeper zookeeper;
    private String lockPath = "/distributed_lock";
    
    public boolean acquireLock(String resourceName) throws Exception {
        String path = lockPath + "/" + resourceName;
        try {
            zookeeper.create(path, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, 
                           CreateMode.EPHEMERAL);
            return true;
        } catch (KeeperException.NodeExistsException e) {
            return false;
        }
    }
    
    public void releaseLock(String resourceName) throws Exception {
        String path = lockPath + "/" + resourceName;
        zookeeper.delete(path, -1);
    }
}
```

### 4. etcd를 이용한 분산락

```go
// etcd를 이용한 분산락 구현 예제 (Go)
func acquireLock(client *clientv3.Client, lockName string, ttl int64) (*clientv3.LeaseGrantResponse, error) {
    // 임대(lease) 생성
    lease, err := client.Grant(context.Background(), ttl)
    if err != nil {
        return nil, err
    }
    
    // 키-값 쌍 설정 시도 (락 획득)
    _, err = client.Txn(context.Background()).
        If(clientv3.Compare(clientv3.CreateRevision(lockName), "=", 0)).
        Then(clientv3.OpPut(lockName, "locked", clientv3.WithLease(lease.ID))).
        Commit()
    
    if err != nil {
        return nil, err
    }
    
    return lease, nil
}

func releaseLock(client *clientv3.Client, lease *clientv3.LeaseGrantResponse) error {
    _, err := client.Revoke(context.Background(), lease.ID)
    return err
}
```

## 분산락 구현 시 고려사항

1. **데드락 방지**: TTL(Time To Live)을 설정하여 락이 무한정 유지되는 것을 방지
2. **재진입성(Reentrant)**: 같은 프로세스가 이미 획득한 락을 다시 획득할 수 있는지 여부
3. **페일오버 처리**: 락을 소유한 서버가 다운되었을 때의 복구 메커니즘
4. **성능**: 락 획득/해제의 성능 및 시스템 부하 고려
5. **락 획득 실패 시 전략**: 대기, 재시도, 폴백 등의 전략

## 분산락 알고리즘

- **Redlock**: Redis를 사용한 분산락 알고리즘으로, 여러 Redis 인스턴스를 사용해 고가용성 보장
- **Chubby**: Google에서 개발한 분산 락 서비스
- **Fencing Token**: 락 해제 후에도 이전 락 소유자의 작업을 방지하기 위한 토큰 메커니즘

분산락은 마이크로서비스 아키텍처, 클라우드 환경, 그리고 다중 서버 환경에서 데이터 일관성과 안정성을 보장하는 중요한 기술입니다. 구현 방식은 시스템 요구사항과 인프라 환경에 따라 선택할 수 있습니다.
