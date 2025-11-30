# Server Orchestration Specification - Plunderstorm Mobile

**Document ID:** BACK-005
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [System Architecture](#2-system-architecture)
3. [Server Fleet Management](#3-server-fleet-management)
4. [Auto-Scaling System](#4-auto-scaling-system)
5. [Server Allocation](#5-server-allocation)
6. [Regional Management](#6-regional-management)
7. [Health Monitoring](#7-health-monitoring)
8. [Rolling Updates](#8-rolling-updates)
9. [Fleet Dashboard](#9-fleet-dashboard)
10. [Alerting System](#10-alerting-system)
11. [API Specification](#11-api-specification)
12. [Disaster Recovery](#12-disaster-recovery)

---

## 1. Executive Summary

### 1.1 Purpose

This document specifies the Server Orchestration system for Plunderstorm Mobile, providing dynamic scaling, allocation, and management of the dedicated game server fleet across multiple regions.

### 1.2 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Availability | 99.9% server availability | Critical |
| Responsiveness | Server allocation < 5 seconds | Critical |
| Cost Efficiency | Scale down during low traffic | High |
| Zero Downtime | Rolling updates without interruption | High |
| Observability | Full visibility into fleet status | High |

### 1.3 Key Metrics

| Metric | Target | Maximum |
|--------|--------|---------|
| Server Allocation Time | < 3s | 5s |
| Scale-Up Time | < 60s | 120s |
| Scale-Down Time | < 300s | 600s |
| Fleet Utilization | 60-80% | 90% |
| Failed Allocations | < 0.1% | 0.5% |

### 1.4 System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATION SERVICE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Fleet      │  │   Scaling    │  │  Allocation  │              │
│  │   Manager    │  │   Engine     │  │   Service    │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                 │                 │                       │
│  ┌──────▼─────────────────▼─────────────────▼──────┐               │
│  │              Orchestration Core                  │               │
│  │  • State Management  • Decision Engine          │               │
│  │  • Event Processing  • Metrics Collection       │               │
│  └──────────────────────┬──────────────────────────┘               │
│                         │                                           │
└─────────────────────────┼───────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│   NA-EAST     │ │   EU-WEST     │ │   ASIA-EAST   │
│   Region      │ │   Region      │ │   Region      │
├───────────────┤ ├───────────────┤ ├───────────────┤
│ ┌───┐ ┌───┐  │ │ ┌───┐ ┌───┐  │ │ ┌───┐ ┌───┐  │
│ │VM │ │VM │  │ │ │VM │ │VM │  │ │ │VM │ │VM │  │
│ └───┘ └───┘  │ │ └───┘ └───┘  │ │ └───┘ └───┘  │
│ ┌───┐ ┌───┐  │ │ ┌───┐ ┌───┐  │ │ ┌───┐        │
│ │VM │ │VM │  │ │ │VM │ │VM │  │ │ │VM │        │
│ └───┘ └───┘  │ │ └───┘ └───┘  │ │ └───┘        │
└───────────────┘ └───────────────┘ └───────────────┘
```

---

## 2. System Architecture

### 2.1 Core Components

```
┌─────────────────────────────────────────────────────────────────────┐
│                 ORCHESTRATION SERVICE ARCHITECTURE                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                      API Gateway                             │   │
│  │  • REST API  • gRPC API  • WebSocket (Dashboard)            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌───────────────────────────┼───────────────────────────────────┐ │
│  │                    Service Layer                               │ │
│  │ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌───────────┐│ │
│  │ │   Fleet     │ │  Scaling    │ │ Allocation  │ │  Health   ││ │
│  │ │   Manager   │ │  Engine     │ │  Service    │ │  Monitor  ││ │
│  │ └─────────────┘ └─────────────┘ └─────────────┘ └───────────┘│ │
│  │ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌───────────┐│ │
│  │ │   Update    │ │   Alert     │ │  Metrics    │ │  Region   ││ │
│  │ │   Manager   │ │  Manager    │ │  Collector  │ │  Manager  ││ │
│  │ └─────────────┘ └─────────────┘ └─────────────┘ └───────────┘│ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              │                                      │
│  ┌───────────────────────────┼───────────────────────────────────┐ │
│  │                    Data Layer                                  │ │
│  │ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐              │ │
│  │ │   State     │ │   Time      │ │   Config    │              │ │
│  │ │   Store     │ │   Series    │ │   Store     │              │ │
│  │ │  (Redis)    │ │ (InfluxDB)  │ │   (etcd)    │              │ │
│  │ └─────────────┘ └─────────────┘ └─────────────┘              │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Data Models

```python
# Server Instance Model
@dataclass
class ServerInstance:
    instance_id: str
    vm_id: str
    region: str
    status: ServerStatus  # STARTING, READY, ALLOCATED, DRAINING, TERMINATED
    current_match_id: Optional[str]
    ip_address: str
    port: int
    started_at: datetime
    last_heartbeat: datetime
    metrics: ServerMetrics
    version: str

# Server Status Enum
class ServerStatus(Enum):
    STARTING = "starting"      # VM booting, server initializing
    READY = "ready"            # Available for allocation
    ALLOCATED = "allocated"    # Assigned to a match
    IN_MATCH = "in_match"      # Match in progress
    DRAINING = "draining"      # Finishing match, won't accept new
    TERMINATING = "terminating" # Shutting down
    TERMINATED = "terminated"   # Stopped
    UNHEALTHY = "unhealthy"    # Failed health checks

# VM Instance Model
@dataclass
class VMInstance:
    vm_id: str
    cloud_provider: str  # aws, gcp, azure
    region: str
    instance_type: str
    ip_address: str
    status: VMStatus
    server_capacity: int  # Max servers per VM
    active_servers: int
    started_at: datetime
    cost_per_hour: float

# Regional Pool Model
@dataclass
class RegionalPool:
    region_id: str
    display_name: str
    cloud_provider: str
    cloud_region: str
    min_instances: int
    max_instances: int
    target_utilization: float
    current_vms: int
    current_servers: int
    available_servers: int
    queue_depth: int
```

### 2.3 State Management

```python
class OrchestrationState:
    """Central state management for orchestration"""

    def __init__(self, redis_client: Redis):
        self.redis = redis_client

    async def get_server(self, instance_id: str) -> Optional[ServerInstance]:
        data = await self.redis.hgetall(f"server:{instance_id}")
        return ServerInstance.from_dict(data) if data else None

    async def set_server(self, server: ServerInstance):
        await self.redis.hset(
            f"server:{server.instance_id}",
            mapping=server.to_dict()
        )
        # Add to regional index
        await self.redis.sadd(
            f"region:{server.region}:servers",
            server.instance_id
        )

    async def get_available_servers(self, region: str) -> List[ServerInstance]:
        server_ids = await self.redis.smembers(f"region:{region}:servers")
        servers = []
        for sid in server_ids:
            server = await self.get_server(sid)
            if server and server.status == ServerStatus.READY:
                servers.append(server)
        return servers

    async def get_regional_stats(self, region: str) -> RegionalStats:
        servers = await self.redis.smembers(f"region:{region}:servers")
        stats = RegionalStats(region=region)

        for sid in servers:
            server = await self.get_server(sid)
            if server:
                stats.total_servers += 1
                if server.status == ServerStatus.READY:
                    stats.available_servers += 1
                elif server.status in [ServerStatus.ALLOCATED, ServerStatus.IN_MATCH]:
                    stats.active_servers += 1

        return stats
```

---

## 3. Server Fleet Management

### 3.1 Fleet Manager

```python
class FleetManager:
    """Manages the overall server fleet"""

    def __init__(self, config: FleetConfig):
        self.config = config
        self.state = OrchestrationState()
        self.cloud_providers = {
            'aws': AWSProvider(),
            'gcp': GCPProvider(),
            'azure': AzureProvider()
        }

    async def initialize(self):
        """Initialize fleet state from cloud providers"""
        for region_config in self.config.regions:
            provider = self.cloud_providers[region_config.provider]

            # Discover existing VMs
            vms = await provider.list_instances(
                region=region_config.cloud_region,
                tags={'role': 'game-server'}
            )

            for vm in vms:
                await self.register_vm(vm, region_config.region_id)

    async def register_vm(self, vm: VMInstance, region: str):
        """Register a VM and its server instances"""
        await self.state.set_vm(vm)

        # Query server manager on VM for active servers
        servers = await self.query_vm_servers(vm)
        for server in servers:
            server.region = region
            await self.state.set_server(server)

    async def provision_vm(self, region: str) -> VMInstance:
        """Provision a new VM in the specified region"""
        region_config = self.config.get_region(region)
        provider = self.cloud_providers[region_config.provider]

        # Launch VM
        vm = await provider.launch_instance(
            region=region_config.cloud_region,
            instance_type=region_config.instance_type,
            image=region_config.server_image,
            user_data=self.generate_user_data(region_config)
        )

        # Wait for VM to be ready
        await self.wait_for_vm_ready(vm, timeout=120)

        # Register VM
        await self.register_vm(vm, region)

        self.metrics.record_vm_provisioned(region)
        return vm

    async def terminate_vm(self, vm_id: str):
        """Terminate a VM and its servers"""
        vm = await self.state.get_vm(vm_id)
        if not vm:
            raise VMNotFoundError(vm_id)

        # Drain all servers on VM first
        servers = await self.state.get_servers_on_vm(vm_id)
        for server in servers:
            if server.status not in [ServerStatus.TERMINATED, ServerStatus.READY]:
                await self.drain_server(server.instance_id)

        # Wait for all servers to finish
        await self.wait_for_servers_drained(vm_id, timeout=600)

        # Terminate VM
        provider = self.cloud_providers[vm.cloud_provider]
        await provider.terminate_instance(vm.vm_id)

        # Update state
        await self.state.remove_vm(vm_id)
        self.metrics.record_vm_terminated(vm.region)

    def generate_user_data(self, region_config: RegionConfig) -> str:
        """Generate cloud-init user data for VM"""
        return f"""#!/bin/bash
# Install and configure server manager
curl -sSL {self.config.server_manager_url}/install.sh | bash

# Configure
cat > /etc/game-server/config.json <<EOF
{{
    "orchestrator_url": "{self.config.orchestrator_url}",
    "region": "{region_config.region_id}",
    "servers_per_vm": {region_config.servers_per_vm},
    "server_port_start": 7777,
    "metrics_port": 9090,
    "health_port": 8080
}}
EOF

# Start server manager
systemctl enable game-server-manager
systemctl start game-server-manager
"""
```

### 3.2 Server Lifecycle

```
┌──────────────────────────────────────────────────────────────────┐
│                   SERVER LIFECYCLE                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────┐                                                    │
│  │ STARTING │ ─── VM boots, server process starts                │
│  └────┬─────┘                                                    │
│       │ Health check passes                                      │
│       ▼                                                          │
│  ┌──────────┐                                                    │
│  │  READY   │ ◀─── Available for match allocation               │
│  └────┬─────┘      │                                            │
│       │            │ Match ends                                  │
│       │ Allocated  │                                             │
│       ▼            │                                             │
│  ┌──────────┐      │                                            │
│  │ALLOCATED │ ─────┤ Players connecting                         │
│  └────┬─────┘      │                                            │
│       │            │                                             │
│       │ Match starts                                             │
│       ▼            │                                             │
│  ┌──────────┐      │                                            │
│  │ IN_MATCH │ ─────┘ Match in progress                          │
│  └────┬─────┘                                                    │
│       │                                                          │
│       │ Drain requested (update/scale-down)                      │
│       ▼                                                          │
│  ┌──────────┐                                                    │
│  │ DRAINING │ ─── Finishing current match, no new allocations   │
│  └────┬─────┘                                                    │
│       │                                                          │
│       │ Match complete                                           │
│       ▼                                                          │
│  ┌────────────┐                                                  │
│  │TERMINATING │ ─── Server shutting down                        │
│  └────┬───────┘                                                  │
│       │                                                          │
│       ▼                                                          │
│  ┌────────────┐                                                  │
│  │ TERMINATED │ ─── Server stopped                              │
│  └────────────┘                                                  │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### 3.3 Server Manager (On-VM Agent)

```python
class ServerManager:
    """Agent running on each VM to manage server instances"""

    def __init__(self, config: ServerManagerConfig):
        self.config = config
        self.servers: Dict[str, ServerProcess] = {}
        self.orchestrator = OrchestratorClient(config.orchestrator_url)

    async def start(self):
        """Start the server manager"""
        # Register with orchestrator
        await self.orchestrator.register_vm(
            vm_id=self.get_vm_id(),
            region=self.config.region,
            capacity=self.config.servers_per_vm
        )

        # Start initial server pool
        for i in range(self.config.initial_servers):
            await self.start_server()

        # Start heartbeat loop
        asyncio.create_task(self.heartbeat_loop())

        # Start health check server
        await self.start_health_server()

    async def start_server(self) -> ServerInstance:
        """Start a new game server instance"""
        port = self.get_next_available_port()
        instance_id = self.generate_instance_id()

        # Start server process
        process = await self.spawn_server_process(instance_id, port)

        server = ServerInstance(
            instance_id=instance_id,
            vm_id=self.get_vm_id(),
            region=self.config.region,
            status=ServerStatus.STARTING,
            ip_address=self.get_public_ip(),
            port=port,
            started_at=datetime.utcnow(),
            version=self.config.server_version
        )

        self.servers[instance_id] = ServerProcess(server, process)

        # Wait for server to be ready
        await self.wait_for_server_ready(instance_id)

        # Register with orchestrator
        server.status = ServerStatus.READY
        await self.orchestrator.register_server(server)

        return server

    async def stop_server(self, instance_id: str, graceful: bool = True):
        """Stop a server instance"""
        if instance_id not in self.servers:
            raise ServerNotFoundError(instance_id)

        server_proc = self.servers[instance_id]

        if graceful:
            # Send shutdown signal
            await server_proc.process.send_signal('SIGTERM')
            # Wait for graceful shutdown
            try:
                await asyncio.wait_for(
                    server_proc.process.wait(),
                    timeout=30
                )
            except asyncio.TimeoutError:
                # Force kill
                server_proc.process.kill()
        else:
            server_proc.process.kill()

        # Unregister from orchestrator
        await self.orchestrator.unregister_server(instance_id)

        del self.servers[instance_id]

    async def heartbeat_loop(self):
        """Send periodic heartbeats to orchestrator"""
        while True:
            try:
                servers_status = [
                    {
                        'instance_id': s.server.instance_id,
                        'status': s.server.status.value,
                        'metrics': await self.collect_server_metrics(s)
                    }
                    for s in self.servers.values()
                ]

                await self.orchestrator.heartbeat(
                    vm_id=self.get_vm_id(),
                    servers=servers_status
                )
            except Exception as e:
                logger.error(f"Heartbeat failed: {e}")

            await asyncio.sleep(5)  # Heartbeat every 5 seconds
```

---

## 4. Auto-Scaling System

### 4.1 Scaling Engine

```python
class ScalingEngine:
    """Manages auto-scaling of server fleet"""

    def __init__(self, config: ScalingConfig, fleet_manager: FleetManager):
        self.config = config
        self.fleet = fleet_manager
        self.state = OrchestrationState()

    async def evaluate_scaling(self):
        """Evaluate and execute scaling decisions for all regions"""
        for region in self.config.regions:
            await self.evaluate_region(region)

    async def evaluate_region(self, region: str):
        """Evaluate scaling for a specific region"""
        stats = await self.state.get_regional_stats(region)
        config = self.config.get_region_config(region)

        # Calculate current utilization
        utilization = self.calculate_utilization(stats)

        # Get queue depth (players waiting for match)
        queue_depth = await self.get_queue_depth(region)

        # Make scaling decision
        decision = self.make_scaling_decision(
            stats, config, utilization, queue_depth
        )

        if decision.action == ScalingAction.SCALE_UP:
            await self.scale_up(region, decision.count)
        elif decision.action == ScalingAction.SCALE_DOWN:
            await self.scale_down(region, decision.count)

    def calculate_utilization(self, stats: RegionalStats) -> float:
        """Calculate server utilization percentage"""
        if stats.total_servers == 0:
            return 0.0
        return stats.active_servers / stats.total_servers

    def make_scaling_decision(
        self,
        stats: RegionalStats,
        config: RegionScalingConfig,
        utilization: float,
        queue_depth: int
    ) -> ScalingDecision:
        """Determine scaling action based on metrics"""

        # Scale up conditions
        if utilization > config.scale_up_threshold:
            # High utilization - scale up
            servers_needed = self.calculate_servers_needed(
                stats, config.target_utilization
            )
            vms_needed = math.ceil(servers_needed / config.servers_per_vm)
            return ScalingDecision(
                action=ScalingAction.SCALE_UP,
                count=min(vms_needed, config.max_scale_up_step),
                reason=f"High utilization: {utilization:.1%}"
            )

        if queue_depth > config.queue_threshold:
            # Queue building up - scale up proactively
            servers_needed = math.ceil(queue_depth / 60)  # 60 players per server
            vms_needed = math.ceil(servers_needed / config.servers_per_vm)
            return ScalingDecision(
                action=ScalingAction.SCALE_UP,
                count=min(vms_needed, config.max_scale_up_step),
                reason=f"Queue depth: {queue_depth}"
            )

        # Scale down conditions
        if utilization < config.scale_down_threshold:
            if stats.current_vms > config.min_instances:
                # Low utilization - scale down
                excess_servers = stats.available_servers - config.min_available
                vms_to_remove = excess_servers // config.servers_per_vm
                return ScalingDecision(
                    action=ScalingAction.SCALE_DOWN,
                    count=min(vms_to_remove, config.max_scale_down_step),
                    reason=f"Low utilization: {utilization:.1%}"
                )

        return ScalingDecision(action=ScalingAction.NONE)

    async def scale_up(self, region: str, count: int):
        """Scale up by adding VMs"""
        logger.info(f"Scaling up {region} by {count} VMs")

        tasks = []
        for _ in range(count):
            tasks.append(self.fleet.provision_vm(region))

        results = await asyncio.gather(*tasks, return_exceptions=True)

        successful = sum(1 for r in results if not isinstance(r, Exception))
        failed = count - successful

        self.metrics.record_scale_up(region, successful, failed)

        if failed > 0:
            logger.error(f"Failed to provision {failed} VMs in {region}")

    async def scale_down(self, region: str, count: int):
        """Scale down by removing VMs"""
        logger.info(f"Scaling down {region} by {count} VMs")

        # Select VMs to terminate (prefer least utilized)
        vms = await self.select_vms_for_termination(region, count)

        for vm in vms:
            # Mark for draining
            await self.fleet.drain_vm(vm.vm_id)

        self.metrics.record_scale_down(region, len(vms))

    async def select_vms_for_termination(
        self,
        region: str,
        count: int
    ) -> List[VMInstance]:
        """Select VMs to terminate based on utilization"""
        vms = await self.state.get_vms_in_region(region)

        # Sort by number of active servers (ascending)
        vms.sort(key=lambda v: v.active_servers)

        # Filter out VMs with active matches
        candidates = [
            vm for vm in vms
            if await self.can_terminate_vm(vm)
        ]

        return candidates[:count]
```

### 4.2 Scaling Configuration

```yaml
# scaling_config.yaml
scaling:
  evaluation_interval: 30  # seconds
  cooldown_period: 300     # seconds between scaling actions

  regions:
    na-east:
      min_instances: 5
      max_instances: 100
      target_utilization: 0.70
      scale_up_threshold: 0.80
      scale_down_threshold: 0.40
      queue_threshold: 300  # players
      servers_per_vm: 4
      max_scale_up_step: 5
      max_scale_down_step: 2
      min_available: 10     # minimum ready servers

    eu-west:
      min_instances: 5
      max_instances: 80
      target_utilization: 0.70
      scale_up_threshold: 0.80
      scale_down_threshold: 0.40
      queue_threshold: 300
      servers_per_vm: 4
      max_scale_up_step: 5
      max_scale_down_step: 2
      min_available: 10

    asia-east:
      min_instances: 3
      max_instances: 50
      target_utilization: 0.70
      scale_up_threshold: 0.80
      scale_down_threshold: 0.40
      queue_threshold: 200
      servers_per_vm: 4
      max_scale_up_step: 3
      max_scale_down_step: 1
      min_available: 5

  # Time-based scaling overrides
  schedules:
    - name: "peak_hours_na"
      regions: ["na-east", "na-west"]
      cron: "0 18-23 * * *"  # 6 PM - 11 PM
      timezone: "America/New_York"
      min_instances_override: 20

    - name: "weekend_boost"
      regions: ["*"]
      cron: "0 0 * * 6,0"  # Weekends
      target_utilization_override: 0.60

    - name: "night_reduction"
      regions: ["na-east", "na-west"]
      cron: "0 2-8 * * *"  # 2 AM - 8 AM
      timezone: "America/New_York"
      max_instances_override: 20
```

### 4.3 Predictive Scaling

```python
class PredictiveScaler:
    """Predictive auto-scaling based on historical patterns"""

    def __init__(self, metrics_store: MetricsStore):
        self.metrics = metrics_store
        self.model = self.load_model()

    async def predict_capacity_needed(
        self,
        region: str,
        lookahead_minutes: int = 30
    ) -> int:
        """Predict server capacity needed in the future"""

        # Get historical data
        historical = await self.metrics.get_player_count_history(
            region=region,
            duration_hours=168  # 1 week
        )

        # Get current time features
        now = datetime.utcnow()
        features = {
            'hour_of_day': now.hour,
            'day_of_week': now.weekday(),
            'is_weekend': now.weekday() >= 5,
            'current_players': await self.metrics.get_current_players(region),
            'queue_depth': await self.metrics.get_queue_depth(region),
            'recent_trend': self.calculate_trend(historical, minutes=15)
        }

        # Predict player count
        predicted_players = self.model.predict(features, lookahead_minutes)

        # Convert to server count (60 players per server)
        servers_needed = math.ceil(predicted_players / 60)

        # Add buffer
        servers_needed = int(servers_needed * 1.2)  # 20% buffer

        return servers_needed

    def calculate_trend(
        self,
        historical: List[DataPoint],
        minutes: int
    ) -> float:
        """Calculate recent trend in player count"""
        recent = [p for p in historical if p.timestamp > datetime.utcnow() - timedelta(minutes=minutes)]
        if len(recent) < 2:
            return 0.0

        # Simple linear regression
        x = np.arange(len(recent))
        y = np.array([p.value for p in recent])
        slope, _ = np.polyfit(x, y, 1)

        return slope
```

---

## 5. Server Allocation

### 5.1 Allocation Service

```python
class AllocationService:
    """Handles server allocation for new matches"""

    def __init__(self, state: OrchestrationState):
        self.state = state
        self.allocation_lock = asyncio.Lock()

    async def allocate_server(
        self,
        request: AllocationRequest
    ) -> AllocationResult:
        """Allocate a server for a new match"""

        start_time = time.time()

        try:
            async with self.allocation_lock:
                # Find available server in requested region
                server = await self.find_available_server(request.region)

                if not server:
                    # Try fallback regions
                    for fallback in request.fallback_regions:
                        server = await self.find_available_server(fallback)
                        if server:
                            break

                if not server:
                    return AllocationResult(
                        success=False,
                        error="No servers available",
                        allocation_time=time.time() - start_time
                    )

                # Reserve server
                server.status = ServerStatus.ALLOCATED
                server.current_match_id = request.match_id
                await self.state.set_server(server)

                # Notify server to prepare for match
                await self.prepare_server(server, request)

                allocation_time = time.time() - start_time
                self.metrics.record_allocation(
                    region=server.region,
                    success=True,
                    time=allocation_time
                )

                return AllocationResult(
                    success=True,
                    server=server,
                    allocation_time=allocation_time
                )

        except Exception as e:
            logger.error(f"Allocation failed: {e}")
            return AllocationResult(
                success=False,
                error=str(e),
                allocation_time=time.time() - start_time
            )

    async def find_available_server(
        self,
        region: str
    ) -> Optional[ServerInstance]:
        """Find an available server in the region"""

        servers = await self.state.get_available_servers(region)

        if not servers:
            return None

        # Sort by preference (same VM for efficiency, then by load)
        servers.sort(key=lambda s: (
            s.metrics.cpu_usage if s.metrics else 0,
            s.started_at  # Prefer older servers
        ))

        return servers[0]

    async def prepare_server(
        self,
        server: ServerInstance,
        request: AllocationRequest
    ):
        """Prepare server for incoming match"""

        # Send match configuration to server
        await self.send_to_server(server, PrepareMatchCommand(
            match_id=request.match_id,
            mode=request.game_mode,
            config=request.match_config,
            expected_players=request.player_count
        ))

    async def release_server(self, instance_id: str):
        """Release a server back to the pool"""

        server = await self.state.get_server(instance_id)
        if not server:
            raise ServerNotFoundError(instance_id)

        # Reset server state
        server.status = ServerStatus.READY
        server.current_match_id = None
        await self.state.set_server(server)

        self.metrics.record_server_released(server.region)
```

### 5.2 Allocation Request Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                   ALLOCATION REQUEST FLOW                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. Matchmaking Service → Allocation Request                      │
│     {match_id, region, mode, player_count}                       │
│          │                                                        │
│          ▼                                                        │
│  2. Check available servers in region                             │
│     • Query state store for READY servers                        │
│     • Filter by version compatibility                            │
│          │                                                        │
│          ├─── Servers available ──────────────────┐              │
│          │                                         │              │
│          ▼                                         │              │
│  3a. No servers → Check fallback regions           │              │
│      • Try configured fallback regions            │              │
│      • If still none → Return error               │              │
│          │                                         │              │
│          │◀────────────────────────────────────────┘              │
│          ▼                                                        │
│  4. Reserve server                                                │
│     • Set status to ALLOCATED                                    │
│     • Set match_id                                               │
│     • Update state store                                         │
│          │                                                        │
│          ▼                                                        │
│  5. Prepare server                                                │
│     • Send match configuration                                   │
│     • Server loads map, initializes systems                      │
│          │                                                        │
│          ▼                                                        │
│  6. Return allocation result                                      │
│     {server_ip, server_port, connection_token}                   │
│          │                                                        │
│          ▼                                                        │
│  7. Matchmaking Service → Players                                 │
│     Players connect to allocated server                          │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### 5.3 Allocation Strategies

```python
class AllocationStrategy:
    """Base class for allocation strategies"""

    async def select_server(
        self,
        servers: List[ServerInstance],
        request: AllocationRequest
    ) -> Optional[ServerInstance]:
        raise NotImplementedError

class RoundRobinStrategy(AllocationStrategy):
    """Simple round-robin allocation"""

    def __init__(self):
        self.index = 0

    async def select_server(
        self,
        servers: List[ServerInstance],
        request: AllocationRequest
    ) -> Optional[ServerInstance]:
        if not servers:
            return None
        server = servers[self.index % len(servers)]
        self.index += 1
        return server

class LeastLoadedStrategy(AllocationStrategy):
    """Allocate to least loaded server"""

    async def select_server(
        self,
        servers: List[ServerInstance],
        request: AllocationRequest
    ) -> Optional[ServerInstance]:
        if not servers:
            return None

        return min(servers, key=lambda s: s.metrics.cpu_usage if s.metrics else 0)

class BinPackingStrategy(AllocationStrategy):
    """Pack servers to maximize VM utilization (cost optimization)"""

    async def select_server(
        self,
        servers: List[ServerInstance],
        request: AllocationRequest
    ) -> Optional[ServerInstance]:
        if not servers:
            return None

        # Group by VM
        by_vm: Dict[str, List[ServerInstance]] = {}
        for server in servers:
            by_vm.setdefault(server.vm_id, []).append(server)

        # Prefer VMs that are already partially utilized
        for vm_id in sorted(by_vm.keys(), key=lambda v: -len(by_vm[v])):
            if by_vm[vm_id]:
                return by_vm[vm_id][0]

        return servers[0]
```

---

## 6. Regional Management

### 6.1 Region Manager

```python
class RegionManager:
    """Manages server pools across regions"""

    def __init__(self, config: RegionConfig):
        self.config = config
        self.regions: Dict[str, RegionalPool] = {}

    async def initialize(self):
        """Initialize regional pools"""
        for region_config in self.config.regions:
            pool = RegionalPool(
                region_id=region_config.region_id,
                display_name=region_config.display_name,
                cloud_provider=region_config.provider,
                cloud_region=region_config.cloud_region,
                min_instances=region_config.min_instances,
                max_instances=region_config.max_instances,
                target_utilization=region_config.target_utilization
            )
            self.regions[region_config.region_id] = pool

    async def get_best_region(
        self,
        player_regions: List[str],
        party_size: int
    ) -> str:
        """Determine best region for a group of players"""

        if len(player_regions) == 1:
            return player_regions[0]

        # Calculate region scores based on:
        # 1. Number of players in each region
        # 2. Server availability
        # 3. Current latency

        region_scores: Dict[str, float] = {}

        for region in set(player_regions):
            # Base score from player count
            player_count = player_regions.count(region)
            score = player_count / len(player_regions)

            # Availability bonus
            pool = self.regions.get(region)
            if pool and pool.available_servers > 0:
                score += 0.2

            region_scores[region] = score

        # Return region with highest score
        return max(region_scores.keys(), key=lambda r: region_scores[r])

    async def get_fallback_regions(self, primary_region: str) -> List[str]:
        """Get fallback regions for a primary region"""

        fallback_map = {
            'na-east': ['na-west', 'eu-west'],
            'na-west': ['na-east', 'asia-east'],
            'eu-west': ['na-east', 'eu-north'],
            'eu-north': ['eu-west', 'na-east'],
            'asia-east': ['asia-southeast', 'na-west'],
            'asia-southeast': ['asia-east', 'eu-west']
        }

        return fallback_map.get(primary_region, [])
```

### 6.2 Regional Configuration

```json
{
  "regions": [
    {
      "region_id": "na-east",
      "display_name": "North America East",
      "provider": "aws",
      "cloud_region": "us-east-1",
      "availability_zones": ["us-east-1a", "us-east-1b", "us-east-1c"],
      "instance_type": "c5.2xlarge",
      "server_image": "ami-gameserver-v1.2.0",
      "min_instances": 5,
      "max_instances": 100,
      "servers_per_vm": 4,
      "target_utilization": 0.70
    },
    {
      "region_id": "na-west",
      "display_name": "North America West",
      "provider": "aws",
      "cloud_region": "us-west-2",
      "availability_zones": ["us-west-2a", "us-west-2b"],
      "instance_type": "c5.2xlarge",
      "server_image": "ami-gameserver-v1.2.0",
      "min_instances": 3,
      "max_instances": 50,
      "servers_per_vm": 4,
      "target_utilization": 0.70
    },
    {
      "region_id": "eu-west",
      "display_name": "Europe West",
      "provider": "gcp",
      "cloud_region": "europe-west1",
      "availability_zones": ["europe-west1-b", "europe-west1-c"],
      "instance_type": "c2-standard-8",
      "server_image": "projects/plunderstorm/images/gameserver-v1-2-0",
      "min_instances": 5,
      "max_instances": 80,
      "servers_per_vm": 4,
      "target_utilization": 0.70
    },
    {
      "region_id": "asia-east",
      "display_name": "Asia East",
      "provider": "aws",
      "cloud_region": "ap-northeast-1",
      "availability_zones": ["ap-northeast-1a", "ap-northeast-1c"],
      "instance_type": "c5.2xlarge",
      "server_image": "ami-gameserver-v1.2.0",
      "min_instances": 3,
      "max_instances": 50,
      "servers_per_vm": 4,
      "target_utilization": 0.70
    }
  ]
}
```

---

## 7. Health Monitoring

### 7.1 Health Monitor

```python
class HealthMonitor:
    """Monitors health of all servers and VMs"""

    def __init__(self, state: OrchestrationState):
        self.state = state
        self.unhealthy_threshold = 3  # Failed checks before marking unhealthy
        self.failed_checks: Dict[str, int] = {}

    async def run_health_checks(self):
        """Run health checks on all servers"""
        servers = await self.state.get_all_servers()

        tasks = [self.check_server_health(s) for s in servers]
        results = await asyncio.gather(*tasks, return_exceptions=True)

        for server, result in zip(servers, results):
            if isinstance(result, Exception):
                await self.handle_check_failure(server, str(result))
            elif not result.healthy:
                await self.handle_check_failure(server, result.reason)
            else:
                await self.handle_check_success(server)

    async def check_server_health(
        self,
        server: ServerInstance
    ) -> HealthCheckResult:
        """Check health of a single server"""

        try:
            # HTTP health check
            async with aiohttp.ClientSession() as session:
                url = f"http://{server.ip_address}:8080/health"
                async with session.get(url, timeout=5) as response:
                    if response.status != 200:
                        return HealthCheckResult(
                            healthy=False,
                            reason=f"HTTP {response.status}"
                        )

                    data = await response.json()

                    # Check specific health indicators
                    if data.get('tick_rate', 0) < 15:
                        return HealthCheckResult(
                            healthy=False,
                            reason="Low tick rate"
                        )

                    if data.get('memory_usage', 0) > 0.95:
                        return HealthCheckResult(
                            healthy=False,
                            reason="High memory usage"
                        )

                    return HealthCheckResult(healthy=True)

        except asyncio.TimeoutError:
            return HealthCheckResult(
                healthy=False,
                reason="Health check timeout"
            )
        except Exception as e:
            return HealthCheckResult(
                healthy=False,
                reason=str(e)
            )

    async def handle_check_failure(
        self,
        server: ServerInstance,
        reason: str
    ):
        """Handle a failed health check"""

        self.failed_checks[server.instance_id] = \
            self.failed_checks.get(server.instance_id, 0) + 1

        if self.failed_checks[server.instance_id] >= self.unhealthy_threshold:
            logger.warning(
                f"Server {server.instance_id} marked unhealthy: {reason}"
            )

            # Mark server as unhealthy
            server.status = ServerStatus.UNHEALTHY
            await self.state.set_server(server)

            # Trigger remediation
            await self.remediate_unhealthy_server(server)

            # Alert
            await self.alert_manager.send_alert(
                AlertSeverity.WARNING,
                f"Server {server.instance_id} unhealthy",
                {"reason": reason, "region": server.region}
            )

    async def handle_check_success(self, server: ServerInstance):
        """Handle a successful health check"""
        # Reset failed check counter
        self.failed_checks[server.instance_id] = 0

        # If was unhealthy, mark as ready
        if server.status == ServerStatus.UNHEALTHY:
            server.status = ServerStatus.READY
            await self.state.set_server(server)
            logger.info(f"Server {server.instance_id} recovered")

    async def remediate_unhealthy_server(self, server: ServerInstance):
        """Attempt to remediate an unhealthy server"""

        # If server has active match, try graceful recovery
        if server.current_match_id:
            # Attempt restart of server process
            success = await self.attempt_server_restart(server)
            if success:
                return

            # If restart failed, migrate match if possible
            await self.alert_manager.send_alert(
                AlertSeverity.CRITICAL,
                f"Server {server.instance_id} unrecoverable with active match",
                {"match_id": server.current_match_id}
            )

        # Terminate and replace server
        await self.fleet_manager.terminate_server(server.instance_id)
```

### 7.2 Heartbeat System

```python
class HeartbeatManager:
    """Manages server heartbeats"""

    def __init__(self, state: OrchestrationState):
        self.state = state
        self.heartbeat_timeout = 30  # seconds

    async def process_heartbeat(
        self,
        vm_id: str,
        servers: List[ServerHeartbeat]
    ):
        """Process heartbeat from a VM"""

        vm = await self.state.get_vm(vm_id)
        if not vm:
            logger.warning(f"Heartbeat from unknown VM: {vm_id}")
            return

        # Update VM last seen
        vm.last_heartbeat = datetime.utcnow()
        await self.state.set_vm(vm)

        # Update server states
        for server_hb in servers:
            server = await self.state.get_server(server_hb.instance_id)
            if server:
                server.last_heartbeat = datetime.utcnow()
                server.metrics = ServerMetrics(
                    cpu_usage=server_hb.cpu_usage,
                    memory_usage=server_hb.memory_usage,
                    player_count=server_hb.player_count,
                    tick_rate=server_hb.tick_rate
                )
                await self.state.set_server(server)

    async def check_stale_heartbeats(self):
        """Check for servers with stale heartbeats"""

        servers = await self.state.get_all_servers()
        now = datetime.utcnow()

        for server in servers:
            if server.status in [ServerStatus.TERMINATED, ServerStatus.TERMINATING]:
                continue

            age = (now - server.last_heartbeat).total_seconds()
            if age > self.heartbeat_timeout:
                logger.warning(
                    f"Server {server.instance_id} heartbeat stale ({age}s)"
                )
                await self.health_monitor.handle_check_failure(
                    server,
                    "Heartbeat timeout"
                )
```

---

## 8. Rolling Updates

### 8.1 Update Manager

```python
class UpdateManager:
    """Manages rolling updates of game servers"""

    def __init__(
        self,
        fleet_manager: FleetManager,
        state: OrchestrationState
    ):
        self.fleet = fleet_manager
        self.state = state

    async def start_rolling_update(
        self,
        new_version: str,
        regions: List[str] = None
    ) -> UpdateJob:
        """Start a rolling update to a new server version"""

        # Create update job
        job = UpdateJob(
            job_id=self.generate_job_id(),
            new_version=new_version,
            regions=regions or list(self.fleet.config.regions.keys()),
            status=UpdateStatus.IN_PROGRESS,
            started_at=datetime.utcnow()
        )

        await self.state.set_update_job(job)

        # Start update in background
        asyncio.create_task(self.execute_update(job))

        return job

    async def execute_update(self, job: UpdateJob):
        """Execute the rolling update"""

        try:
            for region in job.regions:
                await self.update_region(job, region)

            job.status = UpdateStatus.COMPLETED
            job.completed_at = datetime.utcnow()

        except Exception as e:
            logger.error(f"Update failed: {e}")
            job.status = UpdateStatus.FAILED
            job.error = str(e)

        await self.state.set_update_job(job)

    async def update_region(self, job: UpdateJob, region: str):
        """Update servers in a single region"""

        servers = await self.state.get_servers_in_region(region)
        old_servers = [s for s in servers if s.version != job.new_version]

        # Update in batches to maintain capacity
        batch_size = max(1, len(old_servers) // 4)  # 25% at a time

        for i in range(0, len(old_servers), batch_size):
            batch = old_servers[i:i + batch_size]

            # Provision new servers first
            new_servers = []
            for _ in batch:
                server = await self.fleet.provision_server(
                    region,
                    version=job.new_version
                )
                new_servers.append(server)

            # Wait for new servers to be ready
            await self.wait_for_servers_ready(new_servers)

            # Drain old servers
            for server in batch:
                await self.fleet.drain_server(server.instance_id)

            # Wait for old servers to finish matches
            await self.wait_for_servers_drained(batch)

            # Terminate old servers
            for server in batch:
                await self.fleet.terminate_server(server.instance_id)

            # Update job progress
            job.servers_updated += len(batch)
            await self.state.set_update_job(job)

            logger.info(
                f"Update {job.job_id}: {job.servers_updated} servers updated"
            )
```

### 8.2 Update Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                   ROLLING UPDATE FLOW                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Initial State: 20 servers running v1.0                          │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Batch 1 (5 servers)                                         ││
│  │                                                              ││
│  │  1. Provision 5 new v1.1 servers                            ││
│  │     [v1.0 x20] + [v1.1 x5 starting]                        ││
│  │                                                              ││
│  │  2. Wait for v1.1 servers ready                             ││
│  │     [v1.0 x20] + [v1.1 x5 ready]                           ││
│  │                                                              ││
│  │  3. Drain 5 v1.0 servers                                    ││
│  │     [v1.0 x15 active, 5 draining] + [v1.1 x5 ready]        ││
│  │                                                              ││
│  │  4. Wait for drained servers to finish matches              ││
│  │     [v1.0 x15 active] + [v1.1 x5 ready]                    ││
│  │                                                              ││
│  │  5. Terminate old servers                                   ││
│  │     [v1.0 x15] + [v1.1 x5]                                 ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                   │
│  Repeat for Batches 2, 3, 4...                                   │
│                                                                   │
│  Final State: 20 servers running v1.1                            │
│                                                                   │
│  Key Properties:                                                  │
│  • Capacity never drops below original                           │
│  • Active matches complete on original version                   │
│  • New matches start on new version                              │
│  • Zero player disruption                                        │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### 8.3 Canary Deployments

```python
class CanaryDeployment:
    """Canary deployment for new server versions"""

    async def start_canary(
        self,
        new_version: str,
        canary_percentage: float = 0.05
    ) -> CanaryJob:
        """Start a canary deployment"""

        job = CanaryJob(
            job_id=self.generate_job_id(),
            new_version=new_version,
            canary_percentage=canary_percentage,
            status=CanaryStatus.IN_PROGRESS
        )

        # Provision canary servers (5% of fleet)
        for region in self.fleet.config.regions:
            region_servers = await self.state.get_servers_in_region(region)
            canary_count = max(1, int(len(region_servers) * canary_percentage))

            for _ in range(canary_count):
                await self.fleet.provision_server(
                    region,
                    version=new_version,
                    tags={'canary': job.job_id}
                )

        # Start monitoring
        asyncio.create_task(self.monitor_canary(job))

        return job

    async def monitor_canary(self, job: CanaryJob):
        """Monitor canary deployment health"""

        monitoring_duration = timedelta(hours=2)
        start_time = datetime.utcnow()

        while datetime.utcnow() - start_time < monitoring_duration:
            # Compare metrics between canary and stable
            canary_metrics = await self.get_version_metrics(job.new_version)
            stable_metrics = await self.get_version_metrics(self.current_version)

            # Check for degradation
            if self.detect_degradation(canary_metrics, stable_metrics):
                logger.warning(f"Canary {job.job_id} showing degradation")
                await self.rollback_canary(job)
                return

            await asyncio.sleep(60)  # Check every minute

        # Canary successful - proceed with full rollout
        logger.info(f"Canary {job.job_id} successful, starting full rollout")
        await self.update_manager.start_rolling_update(job.new_version)

    def detect_degradation(
        self,
        canary: VersionMetrics,
        stable: VersionMetrics
    ) -> bool:
        """Detect if canary is performing worse than stable"""

        # Error rate threshold
        if canary.error_rate > stable.error_rate * 1.5:
            return True

        # Crash rate threshold
        if canary.crash_rate > stable.crash_rate * 2:
            return True

        # Tick rate threshold
        if canary.avg_tick_rate < stable.avg_tick_rate * 0.9:
            return True

        return False
```

---

## 9. Fleet Dashboard

### 9.1 Dashboard API

```python
class DashboardAPI:
    """API for fleet dashboard"""

    def __init__(self, state: OrchestrationState):
        self.state = state

    async def get_fleet_overview(self) -> FleetOverview:
        """Get high-level fleet overview"""

        regions = []
        total_servers = 0
        total_active = 0
        total_available = 0

        for region_id in self.state.get_all_regions():
            stats = await self.state.get_regional_stats(region_id)
            regions.append(stats)
            total_servers += stats.total_servers
            total_active += stats.active_servers
            total_available += stats.available_servers

        return FleetOverview(
            total_servers=total_servers,
            active_servers=total_active,
            available_servers=total_available,
            utilization=total_active / total_servers if total_servers > 0 else 0,
            regions=regions
        )

    async def get_region_details(self, region: str) -> RegionDetails:
        """Get detailed information for a region"""

        stats = await self.state.get_regional_stats(region)
        servers = await self.state.get_servers_in_region(region)
        vms = await self.state.get_vms_in_region(region)

        # Calculate metrics
        avg_cpu = sum(s.metrics.cpu_usage for s in servers if s.metrics) / len(servers) if servers else 0
        avg_memory = sum(s.metrics.memory_usage for s in servers if s.metrics) / len(servers) if servers else 0

        return RegionDetails(
            region_id=region,
            stats=stats,
            servers=servers,
            vms=vms,
            avg_cpu_usage=avg_cpu,
            avg_memory_usage=avg_memory,
            cost_per_hour=sum(vm.cost_per_hour for vm in vms)
        )

    async def get_server_details(self, instance_id: str) -> ServerDetails:
        """Get detailed information for a server"""

        server = await self.state.get_server(instance_id)
        if not server:
            raise ServerNotFoundError(instance_id)

        # Get match info if allocated
        match_info = None
        if server.current_match_id:
            match_info = await self.get_match_info(server.current_match_id)

        # Get recent metrics history
        metrics_history = await self.metrics.get_server_metrics_history(
            instance_id,
            duration_minutes=60
        )

        return ServerDetails(
            server=server,
            match_info=match_info,
            metrics_history=metrics_history
        )
```

### 9.2 Dashboard Views

```typescript
// Dashboard data structures (TypeScript)

interface FleetOverview {
    totalServers: number;
    activeServers: number;
    availableServers: number;
    utilization: number;
    regions: RegionalStats[];
    alerts: Alert[];
    recentEvents: Event[];
}

interface RegionalStats {
    regionId: string;
    displayName: string;
    totalServers: number;
    activeServers: number;
    availableServers: number;
    unhealthyServers: number;
    queueDepth: number;
    avgLatency: number;
    utilization: number;
    status: 'healthy' | 'degraded' | 'critical';
}

interface ServerListItem {
    instanceId: string;
    vmId: string;
    status: ServerStatus;
    version: string;
    matchId: string | null;
    playerCount: number;
    cpuUsage: number;
    memoryUsage: number;
    uptime: number;
}

interface VMListItem {
    vmId: string;
    provider: string;
    instanceType: string;
    status: VMStatus;
    serverCount: number;
    activeServers: number;
    costPerHour: number;
    uptime: number;
}
```

### 9.3 Real-Time Updates

```python
class DashboardWebSocket:
    """WebSocket handler for real-time dashboard updates"""

    def __init__(self, state: OrchestrationState):
        self.state = state
        self.connections: Set[WebSocket] = set()

    async def handle_connection(self, websocket: WebSocket):
        """Handle new WebSocket connection"""

        await websocket.accept()
        self.connections.add(websocket)

        try:
            # Send initial state
            overview = await self.dashboard_api.get_fleet_overview()
            await websocket.send_json({
                'type': 'fleet_overview',
                'data': overview.to_dict()
            })

            # Keep connection alive and handle messages
            while True:
                message = await websocket.receive_json()
                await self.handle_message(websocket, message)

        except WebSocketDisconnect:
            self.connections.remove(websocket)

    async def broadcast_update(self, update_type: str, data: dict):
        """Broadcast update to all connected clients"""

        message = {
            'type': update_type,
            'data': data,
            'timestamp': datetime.utcnow().isoformat()
        }

        disconnected = set()
        for ws in self.connections:
            try:
                await ws.send_json(message)
            except:
                disconnected.add(ws)

        self.connections -= disconnected

    async def on_server_status_change(self, server: ServerInstance):
        """Called when server status changes"""
        await self.broadcast_update('server_update', {
            'instance_id': server.instance_id,
            'status': server.status.value,
            'region': server.region
        })

    async def on_scaling_event(self, event: ScalingEvent):
        """Called when scaling occurs"""
        await self.broadcast_update('scaling_event', event.to_dict())
```

---

## 10. Alerting System

### 10.1 Alert Manager

```python
class AlertManager:
    """Manages alerts for the orchestration system"""

    def __init__(self, config: AlertConfig):
        self.config = config
        self.active_alerts: Dict[str, Alert] = {}
        self.notifiers = [
            SlackNotifier(config.slack_webhook),
            PagerDutyNotifier(config.pagerduty_key),
            EmailNotifier(config.email_config)
        ]

    async def send_alert(
        self,
        severity: AlertSeverity,
        title: str,
        details: dict,
        region: str = None
    ):
        """Send an alert"""

        alert = Alert(
            alert_id=self.generate_alert_id(),
            severity=severity,
            title=title,
            details=details,
            region=region,
            created_at=datetime.utcnow()
        )

        # Deduplicate
        dedup_key = f"{title}:{region}"
        if dedup_key in self.active_alerts:
            existing = self.active_alerts[dedup_key]
            if (datetime.utcnow() - existing.created_at).seconds < 300:
                # Suppress duplicate within 5 minutes
                return

        self.active_alerts[dedup_key] = alert

        # Send to notifiers based on severity
        tasks = []
        for notifier in self.notifiers:
            if notifier.should_notify(severity):
                tasks.append(notifier.send(alert))

        await asyncio.gather(*tasks, return_exceptions=True)

        # Store alert
        await self.store_alert(alert)

    async def resolve_alert(self, alert_id: str):
        """Resolve an alert"""

        alert = await self.get_alert(alert_id)
        if alert:
            alert.resolved_at = datetime.utcnow()
            alert.status = AlertStatus.RESOLVED
            await self.store_alert(alert)

            # Remove from active alerts
            dedup_key = f"{alert.title}:{alert.region}"
            self.active_alerts.pop(dedup_key, None)
```

### 10.2 Alert Rules

```yaml
# alert_rules.yaml
alerts:
  - name: high_utilization
    condition: "utilization > 0.90"
    duration: 5m
    severity: warning
    title: "High server utilization in {region}"
    description: "Server utilization is above 90% for 5+ minutes"

  - name: critical_utilization
    condition: "utilization > 0.95"
    duration: 2m
    severity: critical
    title: "Critical server utilization in {region}"
    description: "Server utilization is above 95% - scaling may be failing"

  - name: no_available_servers
    condition: "available_servers == 0"
    duration: 1m
    severity: critical
    title: "No available servers in {region}"
    description: "All servers are in use - new matches cannot start"

  - name: high_allocation_failures
    condition: "allocation_failure_rate > 0.05"
    duration: 5m
    severity: warning
    title: "High allocation failure rate"
    description: "More than 5% of allocation requests are failing"

  - name: unhealthy_servers
    condition: "unhealthy_servers > 3"
    duration: 5m
    severity: warning
    title: "Multiple unhealthy servers in {region}"
    description: "{count} servers are unhealthy"

  - name: scaling_failure
    condition: "scaling_failures > 0"
    duration: 1m
    severity: critical
    title: "Scaling failure in {region}"
    description: "Failed to scale server fleet"

  - name: vm_provisioning_slow
    condition: "avg_vm_provision_time > 120"
    duration: 10m
    severity: warning
    title: "Slow VM provisioning"
    description: "VM provisioning taking longer than expected"

  - name: high_queue_depth
    condition: "queue_depth > 500"
    duration: 2m
    severity: warning
    title: "High matchmaking queue in {region}"
    description: "{count} players waiting for matches"
```

---

## 11. API Specification

### 11.1 REST API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/fleet` | GET | Get fleet overview |
| `/api/v1/regions` | GET | List all regions |
| `/api/v1/regions/{id}` | GET | Get region details |
| `/api/v1/servers` | GET | List servers |
| `/api/v1/servers/{id}` | GET | Get server details |
| `/api/v1/servers/{id}/drain` | POST | Drain server |
| `/api/v1/vms` | GET | List VMs |
| `/api/v1/vms/{id}` | GET | Get VM details |
| `/api/v1/allocate` | POST | Allocate server |
| `/api/v1/release/{id}` | POST | Release server |
| `/api/v1/scale` | POST | Manual scale |
| `/api/v1/updates` | GET | List update jobs |
| `/api/v1/updates` | POST | Start update |
| `/api/v1/updates/{id}` | GET | Get update status |
| `/api/v1/alerts` | GET | List alerts |
| `/api/v1/metrics` | GET | Get metrics |

### 11.2 gRPC API

```protobuf
syntax = "proto3";

package orchestration;

service OrchestrationService {
    // Server allocation
    rpc AllocateServer(AllocateRequest) returns (AllocateResponse);
    rpc ReleaseServer(ReleaseRequest) returns (ReleaseResponse);

    // Server management
    rpc RegisterServer(RegisterServerRequest) returns (RegisterServerResponse);
    rpc UnregisterServer(UnregisterServerRequest) returns (UnregisterServerResponse);
    rpc Heartbeat(HeartbeatRequest) returns (HeartbeatResponse);

    // Fleet queries
    rpc GetFleetStatus(GetFleetStatusRequest) returns (GetFleetStatusResponse);
    rpc GetRegionStatus(GetRegionStatusRequest) returns (GetRegionStatusResponse);

    // Streaming updates
    rpc StreamUpdates(StreamUpdatesRequest) returns (stream FleetUpdate);
}

message AllocateRequest {
    string match_id = 1;
    string region = 2;
    repeated string fallback_regions = 3;
    string game_mode = 4;
    int32 expected_players = 5;
    map<string, string> match_config = 6;
}

message AllocateResponse {
    bool success = 1;
    string error = 2;
    ServerInfo server = 3;
    string connection_token = 4;
    float allocation_time_ms = 5;
}

message ServerInfo {
    string instance_id = 1;
    string ip_address = 2;
    int32 port = 3;
    string region = 4;
    string version = 5;
}

message HeartbeatRequest {
    string vm_id = 1;
    repeated ServerStatus servers = 2;
}

message ServerStatus {
    string instance_id = 1;
    string status = 2;
    string match_id = 3;
    int32 player_count = 4;
    float cpu_usage = 5;
    float memory_usage = 6;
    float tick_rate = 7;
}
```

---

## 12. Disaster Recovery

### 12.1 Failure Scenarios

| Scenario | Impact | Recovery |
|----------|--------|----------|
| Single server crash | 1 match affected | Auto-restart or terminate |
| VM failure | 4 matches affected | Auto-provision replacement |
| Region outage | Regional matches affected | Failover to other regions |
| Orchestrator failure | No new allocations | Multi-instance HA |
| Database failure | State lost | Redis cluster + backups |

### 12.2 High Availability

```
┌─────────────────────────────────────────────────────────────────┐
│                 ORCHESTRATOR HIGH AVAILABILITY                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │Orchestrator │    │Orchestrator │    │Orchestrator │         │
│  │  Primary    │    │  Secondary  │    │  Secondary  │         │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘         │
│         │                  │                  │                 │
│         └──────────────────┼──────────────────┘                 │
│                            │                                    │
│                    ┌───────▼───────┐                           │
│                    │ Load Balancer │                           │
│                    └───────┬───────┘                           │
│                            │                                    │
│         ┌──────────────────┼──────────────────┐                │
│         │                  │                  │                 │
│         ▼                  ▼                  ▼                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   Redis     │    │   Redis     │    │   Redis     │         │
│  │  Primary    │◀──▶│  Replica    │◀──▶│  Replica    │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 12.3 Backup and Recovery

```python
class DisasterRecovery:
    """Disaster recovery procedures"""

    async def backup_state(self):
        """Backup orchestration state"""

        state_snapshot = {
            'timestamp': datetime.utcnow().isoformat(),
            'servers': await self.state.get_all_servers(),
            'vms': await self.state.get_all_vms(),
            'config': self.config.to_dict()
        }

        # Upload to S3/GCS
        await self.storage.upload(
            f"backups/orchestration/{datetime.utcnow().strftime('%Y%m%d_%H%M%S')}.json",
            json.dumps(state_snapshot)
        )

    async def restore_from_backup(self, backup_path: str):
        """Restore state from backup"""

        data = await self.storage.download(backup_path)
        snapshot = json.loads(data)

        # Restore servers
        for server_data in snapshot['servers']:
            server = ServerInstance.from_dict(server_data)
            # Verify server still exists
            if await self.verify_server_exists(server):
                await self.state.set_server(server)

        # Restore VMs
        for vm_data in snapshot['vms']:
            vm = VMInstance.from_dict(vm_data)
            if await self.verify_vm_exists(vm):
                await self.state.set_vm(vm)

    async def failover_region(self, failed_region: str):
        """Handle region failover"""

        logger.critical(f"Initiating failover for region {failed_region}")

        # Get players in failed region's queue
        queue = await self.matchmaking.get_queue(failed_region)

        # Move to fallback regions
        fallback_regions = await self.region_manager.get_fallback_regions(failed_region)

        for player in queue:
            await self.matchmaking.move_to_region(
                player,
                fallback_regions[0]
            )

        # Alert operations
        await self.alert_manager.send_alert(
            AlertSeverity.CRITICAL,
            f"Region {failed_region} failover initiated",
            {'players_moved': len(queue)}
        )
```

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Backend Lead | _____________ | _____________ | ___/___/___ |
| Infrastructure Lead | _____________ | _____________ | ___/___/___ |
| DevOps Lead | _____________ | _____________ | ___/___/___ |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial server orchestration specification |

---

*This document contains implementation details. Handle according to company security policies.*
