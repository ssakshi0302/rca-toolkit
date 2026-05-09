# Complete Temporal Metrics Catalog by Service

**Source**: Temporal Doctor pd-triage skill (OrcaaS team)  
**URL**: https://git.soma.salesforce.com/orcaas/temporal-doctor/pull/3  
**Purpose**: Authoritative reference for all Temporal metrics available via Argus

---

## Temporal Official References

### Source of Truth — Temporal Server Source Code
- **GitHub repo**: https://github.com/temporalio/temporal
  - This is the canonical source for all metrics definitions, error messages, and server behavior.
  - When Splunk logs show an unfamiliar error message, search this repo to find the exact code
    path that produces it, understand the conditions that trigger it, and identify the fix.
  - Key source paths for metrics: `common/metrics/`, `service/frontend/`, `service/history/`,
    `service/matching/`, `service/worker/`
  - Key source paths for errors: `common/rpc/interceptor/`, `service/*/api/`

### Documentation
- **All references index**: https://docs.temporal.io/references/
- **Cluster metrics**: https://docs.temporal.io/references/cluster-metrics
- **Server errors**: https://docs.temporal.io/references/errors
- **Events**: https://docs.temporal.io/references/events
- **Commands**: https://docs.temporal.io/references/commands
- **SDK metrics**: https://docs.temporal.io/references/sdk-metrics
- **Configuration**: https://docs.temporal.io/references/configuration
- **Dynamic configuration**: https://docs.temporal.io/references/dynamic-configuration
- **Troubleshooting guide**: https://docs.temporal.io/troubleshooting/

### Falcon Infrastructure Metrics
- **Out-of-the-box metrics spreadsheet**: https://docs.google.com/spreadsheets/d/1yFKxuPHrwl14D4kA91rt27x8x44AjllgBBpXzzXqxv8
  - cAdvisor, kube-state-metrics, kubelet, cluster-autoscaler, kube-apiserver, collectD

Use these to look up metric definitions, error codes/meanings, event types, configuration,
and known troubleshooting patterns when triaging incidents. When Splunk logs show a specific
error message, first check the Temporal GitHub repo source code to understand the exact code
path, then the errors reference for documented meanings, and the troubleshooting guide for
known resolution patterns.

---

## How to Query These Metrics

**Argus scope pattern**: `temporal.<service>.aws.<FI>.<FD>`  
**Service instance filter**: `{service_instance=<SI>}`

Example:
```
temporal.temporalfrontend.aws.esvc1-useast2.foundation
{service_instance=temporalfrontend2001}
```

---

## Metric Type Legend

- **C** = Counter (monotonically increasing, use RATE transform)
- **H** = Histogram (emitted as `_sum` + `_count` counters, use DIVIDE_V(RATE(_sum), RATE(_count)) for avg)
- **G** = Gauge (instantaneous value, query directly with avg aggregator)

---

## Frontend Service (`temporalfrontend`)

### Traffic & Errors
| Metric | Type | What It Means |
|--------|------|---------------|
| `client_requests` | C | Incoming gRPC requests from external clients |
| `client_errors` | C | Failed gRPC requests from external clients |
| `client_latency` / `_sum` / `_count` | H | End-to-end latency for client gRPC calls |
| `client_redirection_requests` | C | Requests redirected to another cluster |
| `client_redirection_errors` | C | Failed redirect attempts |
| `client_redirection_latency` / `_sum` / `_count` | H | Latency for redirected requests |
| `request_count_total` | C | Total HTTP requests (non-gRPC) |
| `request_count_failures` | C | Failed HTTP requests |
| `request_duration` / `_sum` / `_count` | H | HTTP request latency |

### Service-to-Service (Frontend → History/Matching)
| Metric | Type | What It Means |
|--------|------|---------------|
| `service_requests` | C | Outbound internal gRPC calls to other Temporal services |
| `service_errors` | C | Failed internal service calls |
| `service_errors_resource_exhausted` | C | Rate-limited by downstream service |
| `service_errors_nondeterministic` | C | Nondeterministic workflow errors |
| `service_error_with_type` | C | Errors broken down by type (use `error_type` tag) |
| `service_latency` / `_sum` / `_count` | H | Internal service call latency |
| `service_latency_nouserlatency` / `_sum` / `_count` | H | Latency excluding user code time |
| `service_latency_userlatency` / `_sum` / `_count` | H | User code latency portion |
| `service_pending_requests` | G | Pending outbound requests — high = downstream slow |
| `service_dial_success` | C | Successful gRPC dial connections |
| `service_dial_latency` / `_sum` / `_count` | H | gRPC connection establishment latency |
| `service_authorization_latency` / `_sum` / `_count` | H | Auth check latency |

### gRPC Connections
| Metric | Type | What It Means |
|--------|------|---------------|
| `service_grpc_conn_accepted` | C | New gRPC connections accepted |
| `service_grpc_conn_active` | G | Currently active gRPC connections |
| `service_grpc_conn_closed` | C | gRPC connections closed |
| `host_rps_limit` | G | Host-level rate limit setting |

### Persistence (Frontend's DB access)
| Metric | Type | What It Means |
|--------|------|---------------|
| `persistence_requests` | C | DB operations per minute |
| `persistence_errors` | C | DB operation failures |
| `persistence_error_with_type` | C | DB errors by type |
| `persistence_latency` / `_sum` / `_count` | H | DB operation latency |
| `persistence_sql_open_conn` | G | Open SQL connections |
| `persistence_sql_in_use` | G | SQL connections in use |
| `persistence_sql_idle_conn` | G | Idle SQL connections |
| `persistence_sql_max_open_conn` | G | Max open connections setting |
| `persistence_session_refresh_attempts` | C | Session refresh attempts |

### Visibility (Elasticsearch)
| Metric | Type | What It Means |
|--------|------|---------------|
| `visibility_persistence_requests` | C | Visibility store operations |
| `visibility_persistence_errors` | C | Visibility store failures |
| `visibility_persistence_latency` / `_sum` / `_count` | H | Visibility store latency |
| `visibility_persistence_error_with_type` | C | Visibility errors by type |

### Caching
| Metric | Type | What It Means |
|--------|------|---------------|
| `cache_size` | G | Current cache size |
| `cache_ttl` / `_sum` / `_count` | H | Cache entry TTL distribution |

### Runtime
| Metric | Type | What It Means |
|--------|------|---------------|
| `num_goroutines` / `go_goroutines` | G | Go goroutine count — spike = leak |
| `memory_heap` | G | Heap memory usage |
| `memory_heapinuse` | G | Heap memory in active use |
| `memory_allocated` | G | Total allocated memory |
| `memory_stack` | G | Stack memory |
| `process_resident_memory_bytes` | G | RSS memory |
| `process_cpu_seconds_total` | C | CPU usage |
| `restarts` | C | Pod restart counter |
| `up` | G | Instance up/down (1 or 0) |

---

## Matching Service (`temporalmatching`)

### Traffic & Errors
| Metric | Type | What It Means |
|--------|------|---------------|
| `client_requests` | C | Incoming gRPC requests |
| `client_errors` | C | Failed gRPC requests |
| `client_latency` / `_sum` / `_count` | H | Client request latency |
| `service_requests` | C | Internal service calls |
| `service_errors` | C | Internal service failures |
| `service_errors_resource_exhausted` | C | Rate-limited errors |
| `service_error_with_type` | C | Errors by type |
| `service_latency` / `_sum` / `_count` | H | Internal service latency |
| `condition_failed_errors` | C | Optimistic lock failures |

### Task Queue Health (Critical for Matching)
| Metric | Type | What It Means |
|--------|------|---------------|
| `approximate_backlog_count` | G | Tasks waiting in queue — high = consumers can't keep up |
| `approximate_backlog_age_seconds` | G | Age of oldest queued task — high = severe backlog |
| `loaded_task_queue_count` | G | Active task queues — drop = partitions unloaded |
| `loaded_task_queue_family_count` | G | Active task queue families |
| `loaded_task_queue_partition_count` | G | Active task queue partitions |
| `loaded_physical_task_queue_count` | G | Physical task queues loaded |
| `task_queue_started` | C | Task queues started |
| `task_queue_stopped` | C | Task queues stopped |
| `task_lag_per_tl` | G | Per-task-list lag |
| `no_poller_tasks` | C | Tasks with no available poller — workers may be down |

### Polling (Worker ↔ Matching interaction)
| Metric | Type | What It Means |
|--------|------|---------------|
| `poll_success` | C | Tasks successfully matched to pollers |
| `poll_success_sync` | C | Synchronously matched tasks |
| `poll_timeouts` | C | Long-polls that timed out with no task — high = low load or slow dispatch |
| `poll_latency` / `_sum` / `_count` | H | Long-poll duration |

### Task Matching Performance
| Metric | Type | What It Means |
|--------|------|---------------|
| `syncmatch_latency` / `_sum` / `_count` | H | Synchronous match latency (task immediately matched to waiting poller) |
| `asyncmatch_latency` / `_sum` / `_count` | H | Async match latency (task queued, then matched) |
| `task_dispatch_latency` / `_sum` / `_count` | H | Total time from task creation to dispatch |
| `task_write_latency` / `_sum` / `_count` | H | Time to write task to persistence |
| `local_to_local_matches` | C | Tasks matched locally |
| `local_to_remote_matches` | C | Tasks forwarded to remote partition |
| `remote_to_local_matches` | C | Tasks received from remote partition |
| `remote_to_remote_matches` | C | Tasks forwarded between remote partitions |
| `forwarded` | C | Total forwarded tasks |
| `forwarded_per_tl` | C | Forwarded tasks per task list |
| `forward_task_errors` | C | Forwarding failures |

### Task Lifecycle
| Metric | Type | What It Means |
|--------|------|---------------|
| `tasks_expired` | C | Tasks that expired before being matched |
| `task_rewrites` | C | Tasks rewritten (due to version changes) |
| `buffer_throttle_count` | C | Times task buffer was throttled |
| `sync_throttle_count` | C | Times sync matching was throttled |
| `respond_query_failed` | C | Failed query responses |

### Lease Management
| Metric | Type | What It Means |
|--------|------|---------------|
| `lease_requests` | C | Task queue lease requests |
| `lease_failures` | C | Failed lease attempts |

### Persistence
| Metric | Type | What It Means |
|--------|------|---------------|
| `persistence_requests` | C | DB operations |
| `persistence_errors` | C | DB failures |
| `persistence_error_with_type` | C | DB errors by type |
| `persistence_latency` / `_sum` / `_count` | H | DB latency |
| `persistence_sql_open_conn` | G | Open DB connections |
| `persistence_sql_in_use` | G | DB connections in use |
| `persistence_sql_idle_conn` | G | Idle DB connections |
| `persistence_sql_max_open_conn` | G | Max connections setting |

### gRPC & Runtime
| Metric | Type | What It Means |
|--------|------|---------------|
| `service_grpc_conn_accepted` | C | New connections |
| `service_grpc_conn_active` | G | Active connections |
| `service_grpc_conn_closed` | C | Closed connections |
| `service_dial_success` | C | Outbound dial success |
| `service_dial_error` | C | Outbound dial failures |
| `service_dial_latency` / `_sum` / `_count` | H | Dial latency |
| `num_goroutines` | G | Goroutine count |
| `memory_heap` | G | Heap memory |
| `restarts` | C | Pod restarts |
| `up` | G | Up/down |

---

## History Service (`temporalhistory`)

### Traffic & Errors
| Metric | Type | What It Means |
|--------|------|---------------|
| `client_requests` | C | Incoming gRPC requests |
| `client_errors` | C | Failed gRPC requests |
| `client_latency` / `_sum` / `_count` | H | Client request latency |
| `service_requests` | C | Internal service calls |
| `service_errors` | C | Internal service failures |
| `service_errors_resource_exhausted` | C | Rate-limited errors |
| `service_error_with_type` | C | Errors by type |
| `service_latency` / `_sum` / `_count` | H | Internal service latency |

### Workflow Execution
| Metric | Type | What It Means |
|--------|------|---------------|
| `activity_success` | C | Activities completed successfully |
| `activity_fail` | C | Activity failures |
| `activity_timeout` | C | Activity timeouts |
| `activity_task_fail` | C | Activity task processing failures |
| `activity_task_timeout` | C | Activity task timeouts |
| `heartbeat_timeout` | C | Activity heartbeat timeouts |
| `activity_end_to_end_latency` / `_sum` / `_count` | H | Full activity lifecycle latency |
| `activity_schedule_to_close_latency` / `_sum` / `_count` | H | Schedule to close latency |
| `activity_start_to_close_latency` / `_sum` / `_count` | H | Start to close latency |
| `failed_workflow_tasks` | C | Failed workflow task executions |
| `state_transition_count` / `_sum` / `_count` | H | State transitions per workflow |
| `command` | C | Workflow commands processed |

### Shard Management
| Metric | Type | What It Means |
|--------|------|---------------|
| `numshards_gauge` | G | Number of shards owned by this instance |
| `acquire_shards_count` | C | Shard acquisitions |
| `acquire_shards_latency` / `_sum` / `_count` | H | Shard acquisition latency |
| `shard_closed_count` | C | Shards closed/released |
| `membership_changed_count` | C | Cluster membership changes |

### Task Processing (Internal History Tasks)
| Metric | Type | What It Means |
|--------|------|---------------|
| `task_requests` | C | Internal task requests |
| `task_errors` | C | Internal task failures |
| `task_errors_throttled` | C | Throttled task errors |
| `task_latency` / `_sum` / `_count` | H | Task processing latency |
| `task_latency_load` / `_sum` / `_count` | H | Task load latency |
| `task_latency_schedule` / `_sum` / `_count` | H | Task schedule latency |
| `task_latency_processing` / `_sum` / `_count` | H | Task processing-only latency |
| `task_latency_queue` / `_sum` / `_count` | H | Task queue wait latency |
| `task_schedule_to_start_latency` / `_sum` / `_count` | H | Schedule to start delay |
| `pending_tasks` / `_sum` / `_count` | H | Pending task count |
| `task_count` / `_sum` / `_count` | H | Total task count |

### Persistence
| Metric | Type | What It Means |
|--------|------|---------------|
| `persistence_requests` | C | DB operations |
| `persistence_errors` | C | DB failures |
| `persistence_errors_resource_exhausted` | C | DB rate-limited errors |
| `persistence_error_with_type` | C | DB errors by type |
| `persistence_latency` / `_sum` / `_count` | H | DB latency |
| `persistence_shard_rps` / `_sum` / `_count` | H | Per-shard request rate |
| `persistence_sql_open_conn` | G | Open DB connections |
| `persistence_sql_in_use` | G | DB connections in use |

### History & Event Storage
| Metric | Type | What It Means |
|--------|------|---------------|
| `history_size` / `_sum` / `_count` | H | Workflow history size |
| `history_count` / `_sum` / `_count` | H | Workflow history event count |
| `event_blob_size` / `_sum` / `_count` | H | Event blob size |
| `mutable_state_size` / `_sum` / `_count` | H | Workflow mutable state size |
| `execution_info_size` / `_sum` / `_count` | H | Execution info size |
| `buffered_events_count` / `_sum` / `_count` | H | Buffered events |

### Workflow Vital Signs (Direct User Impact Metrics)
| Metric | Type | What It Means |
|--------|------|---------------|
| `workflow_success` | C | Workflows completing successfully — steady = healthy |
| `workflow_failed` | C | Workflow failures — spike = real user impact |
| `workflow_timeout` | C | Timed-out workflows — non-zero = severe |
| `workflow_cancel` | C | Cancelled workflows |
| `workflow_terminate` | C | Terminated workflows — spike = manual cleanup or stuck workflows |
| `workflow_continued_as_new` | C | Long-running workflow rollovers |
| `workflow_tasks_completed` | C | Task throughput |

### Caching
| Metric | Type | What It Means |
|--------|------|---------------|
| `cache_requests` | C | Cache lookups |
| `cache_errors` | C | Cache failures |
| `cache_miss` | C | Cache misses |
| `cache_size` | G | Current cache size |
| `cache_usage` | G | Cache utilization |
| `cache_latency` / `_sum` / `_count` | H | Cache operation latency |

### Elasticsearch (Visibility)
| Metric | Type | What It Means |
|--------|------|---------------|
| `elasticsearch_bulk_processor_requests` | C | ES bulk requests |
| `elasticsearch_bulk_processor_request_latency` / `_sum` / `_count` | H | ES request latency |
| `elasticsearch_bulk_processor_bulk_size` / `_sum` / `_count` | H | ES bulk size |
| `elasticsearch_bulk_processor_queued_requests` / `_sum` / `_count` | H | ES queued requests |

### Replication
| Metric | Type | What It Means |
|--------|------|---------------|
| `replication_tasks_lag` / `_sum` / `_count` | H | Replication lag |
| `replication_task_cleanup_count` | C | Replication task cleanup |

---

## Worker Service (`temporalworker`)

### Traffic & Errors
| Metric | Type | What It Means |
|--------|------|---------------|
| `client_requests` | C | Incoming gRPC requests |
| `client_errors` | C | Failed gRPC requests |
| `client_latency` / `_sum` / `_count` | H | Client request latency |

### SDK Worker Metrics
| Metric | Type | What It Means |
|--------|------|---------------|
| `temporal_request` | C | SDK requests to server |
| `temporal_request_failure` | C | SDK request failures |
| `temporal_request_attempt` | C | SDK request attempts (includes retries) |
| `temporal_request_latency` / `_sum` / `_count` | H | SDK request latency |
| `temporal_request_resource_exhausted` | C | Rate-limited SDK requests |
| `temporal_worker_start` | C | Worker starts |
| `temporal_worker_task_slots_available` | G | Available task execution slots |
| `temporal_worker_task_slots_used` | G | Task slots in use |
| `temporal_num_pollers` | G | Active pollers |
| `temporal_poller_start` | C | Poller starts |

### Workflow Execution
| Metric | Type | What It Means |
|--------|------|---------------|
| `temporal_workflow_completed` | C | Workflows completed |
| `temporal_workflow_continue_as_new` | C | Workflows continued as new |
| `temporal_workflow_task_execution_failed` | C | Failed workflow task executions |
| `temporal_workflow_task_execution_latency` / `_sum` / `_count` | H | Workflow task execution latency |
| `temporal_workflow_task_replay_latency` / `_sum` / `_count` | H | Workflow history replay latency |
| `temporal_workflow_task_schedule_to_start_latency` / `_sum` / `_count` | H | Queue wait time for workflow tasks |
| `temporal_workflow_task_queue_poll_succeed` | C | Successful workflow task polls |
| `temporal_workflow_task_queue_poll_empty` | C | Empty workflow task polls |
| `temporal_workflow_endtoend_latency` / `_sum` / `_count` | H | Full workflow execution latency |
| `temporal_sticky_cache_hit` | C | Sticky execution cache hits |
| `temporal_sticky_cache_size` | G | Sticky cache size |

### Activity Execution
| Metric | Type | What It Means |
|--------|------|---------------|
| `temporal_activity_execution_failed` | C | Activity execution failures |
| `temporal_activity_execution_latency` / `_sum` / `_count` | H | Activity execution latency |
| `temporal_activity_schedule_to_start_latency` / `_sum` / `_count` | H | Activity queue wait time |
| `temporal_activity_succeed_endtoend_latency` / `_sum` / `_count` | H | Successful activity end-to-end latency |
| `temporal_activity_poll_no_task` | C | Activity polls with no available task |
| `temporal_local_activity_total` | C | Local activities executed |
| `temporal_local_activity_failed` | C | Local activity failures |
| `temporal_local_activity_execution_failed` | C | Local activity execution failures |
| `temporal_local_activity_execution_latency` / `_sum` / `_count` | H | Local activity execution latency |

### System Workflows
| Metric | Type | What It Means |
|--------|------|---------------|
| `batcher_processor_requests` | C | Batch processor requests |
| `scavenger_success` | C | Scavenger successful cleanups |
| `scavenger_errors` | C | Scavenger errors |
| `scavenger_skips` | C | Scavenger skipped items |
| `schedule_action_success` | C | Scheduled actions executed |
| `schedule_missed_catchup_window` | C | Missed schedule windows |
| `schedule_action_delay` / `_sum` / `_count` | H | Schedule action delay |

### Persistence
| Metric | Type | What It Means |
|--------|------|---------------|
| `persistence_requests` | C | DB operations |
| `persistence_errors` | C | DB failures |
| `persistence_errors_resource_exhausted` | C | DB rate-limited errors |
| `persistence_latency` / `_sum` / `_count` | H | DB latency |
| `persistence_sql_open_conn` | G | Open DB connections |
| `persistence_sql_in_use` | G | DB connections in use |

---

## Notes

**Last updated**: 2026-05-09  
**Source**: Temporal Doctor pd-triage skill, OrcaaS team operational knowledge  
**Cross-reference**: See `runbooks/metrics-catalog.md` for RCA-specific findings and gaps identified
