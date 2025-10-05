# Project 2: Kubernetes Auto-Scaler & Monitor

## 🎯 Overview
An intelligent Kubernetes autoscaling system that monitors pod metrics, evaluates resource utilization, and automatically scales deployments based on CPU and memory thresholds. This **standalone version requires NO external dependencies** - it uses only Python's standard library and simulates Kubernetes operations for demonstration and learning purposes.

## ✨ Features

### Core Capabilities
- **Real-Time Monitoring**: Monitor CPU and memory usage across all pods
- **Intelligent Auto-Scaling**: Automatically scale deployments up or down based on metrics
- **Cooldown Periods**: Prevent rapid scaling oscillations with configurable cooldown
- **Smart Thresholds**: Separate thresholds for scale-up and scale-down operations
- **Cluster Resource Tracking**: Monitor total cluster capacity and utilization
- **Scaling History**: Track all scaling decisions with detailed reasoning
- **Multi-Deployment Support**: Manage multiple deployments simultaneously
- **Compliance Limits**: Enforce min/max replica constraints

### Scaling Intelligence
- **CPU-Based Scaling**: Scale based on average CPU utilization
- **Memory-Based Scaling**: Scale based on average memory consumption
- **Conservative Scale-Down**: Only scale down when both CPU and memory are low
- **Aggressive Scale-Up**: Scale up when either CPU or memory is high
- **Gradual Scaling**: Scale by one replica at a time for stability

## 🚀 Technologies Used
- **Python Standard Library Only**: No external dependencies required!
- **dataclasses**: Type-safe data structures
- **enum**: Pod phase and status management
- **datetime**: Timestamp tracking and cooldown management
- **concurrent concepts**: Simulated async operations
- **logging**: Comprehensive operational logging

## 📋 Prerequisites

### System Requirements
- **Python 3.7+** (that's it!)
- No Kubernetes cluster required (uses mock data)
- No kubectl or helm needed
- No external packages to install

## 🔧 Installation & Setup

### Quick Start (2 Steps)

**Step 1: Download the file**
```bash
# Save the code as: k8s_autoscaler.py
```

**Step 2: Run it!**
```bash
python k8s_autoscaler.py
```

No installations, no Kubernetes cluster, no configuration needed!

## 💻 Usage

### Running the Demo
```bash
python k8s_autoscaler.py
```

**Expected Output:**
```
======================================================================
 Kubernetes Auto-Scaler & Monitor - Standalone Demo
======================================================================

📊 Cluster Information:
  Nodes: 3
  Total CPU: 12.0 cores
  Total Memory: 48.0 GB

📦 Deployments in 'default' namespace: 5
  - web-app: 3 replicas
  - api-server: 2 replicas
  - worker-service: 4 replicas
  - cache-service: 2 replicas
  - auth-service: 3 replicas

🔄 Running monitoring cycles...
============================================================
Monitoring Cycle #1
============================================================
INFO - Evaluating 5 deployments in namespace default
INFO - ✓ Scaling applied: web-app from 3 to 4 replicas - High CPU usage: 75.3% > 70.0%
INFO - Scaling cycle complete: 1 actions applied
```

### Using as a Library

#### Example 1: Basic Monitoring
```python
from k8s_autoscaler import KubernetesMonitor, AutoScaler

# Initialize monitor (mock mode)
monitor = KubernetesMonitor(mock_mode=True)

# Get cluster resources
cluster_info = monitor.get_cluster_resources()
print(f"Cluster has {cluster_info['node_count']} nodes")
print(f"Total CPU: {cluster_info['total_cpu_cores']} cores")
print(f"Total Memory: {cluster_info['total_memory_gb']} GB")

# List deployments
deployments = monitor.list_deployments('default')
for deployment in deployments:
    info = monitor.get_deployment_info(deployment)
    print(f"{deployment}: {info.replicas} replicas")
```

#### Example 2: Get Pod Metrics
```python
# Get metrics for all pods in namespace
all_metrics = monitor.get_pod_metrics(namespace='default')

for metric in all_metrics:
    cpu_pct = metric.get_cpu_percentage()
    mem_pct = metric.get_memory_percentage()
    print(f"Pod: {metric.name}")
    print(f"  CPU: {cpu_pct:.1f}% ({metric.cpu_usage_millicores}m)")
    print(f"  Memory: {mem_pct:.1f}% ({metric.memory_usage_mb:.0f}MB)")
    print(f"  Node: {metric.node}")
    print()

# Get metrics for specific deployment
web_app_metrics = monitor.get_pod_metrics(namespace='default', deployment='web-app')
avg_cpu = sum(m.get_cpu_percentage() for m in web_app_metrics) / len(web_app_metrics)
print(f"Web-app average CPU: {avg_cpu:.1f}%")
```

#### Example 3: Manual Scaling
```python
# Scale a deployment manually
success = monitor.scale_deployment(
    name='web-app',
    replicas=5,
    namespace='default'
)

if success:
    print("✓ Deployment scaled successfully")
    
# Verify new replica count
info = monitor.get_deployment_info('web-app')
print(f"Current replicas: {info.replicas}")
```

#### Example 4: Initialize AutoScaler
```python
# Create autoscaler with custom thresholds
autoscaler = AutoScaler(
    monitor=monitor,
    cpu_threshold_up=70.0,      # Scale up if CPU > 70%
    cpu_threshold_down=30.0,    # Scale down if CPU < 30%
    memory_threshold_up=80.0,   # Scale up if Memory > 80%
    memory_threshold_down=40.0, # Scale down if Memory < 40%
    min_replicas=2,             # Never scale below 2
    max_replicas=10,            # Never scale above 10
    cooldown_minutes=5          # Wait 5 min between scaling
)
```

#### Example 5: Evaluate Single Deployment
```python
# Evaluate if deployment needs scaling
decision = autoscaler.evaluate_deployment('web-app', namespace='default')

if decision:
    print(f"Scaling Decision for {decision.deployment}:")
    print(f"  Current replicas: {decision.current_replicas}")
    print(f"  Target replicas: {decision.target_replicas}")
    print(f"  Reason: {decision.reason}")
    print(f"  CPU Usage: {decision.cpu_usage_percent:.1f}%")
    print(f"  Memory Usage: {decision.memory_usage_percent:.1f}%")
    
    # Apply the decision
    if autoscaler.apply_scaling_decision(decision):
        print("✓ Scaling applied successfully")
else:
    print("No scaling needed")
```

#### Example 6: Evaluate All Deployments
```python
# Evaluate all deployments at once
decisions = autoscaler.evaluate_all_deployments(namespace='default')

print(f"Found {len(decisions)} scaling recommendations")

for decision in decisions:
    action = "Scale UP" if decision.target_replicas > decision.current_replicas else "Scale DOWN"
    print(f"\n{action}: {decision.deployment}")
    print(f"  {decision.current_replicas} → {decision.target_replicas} replicas")
    print(f"  Reason: {decision.reason}")
```

#### Example 7: Run Single Scaling Cycle
```python
# Run one complete scaling cycle
actions_taken = autoscaler.run_scaling_cycle(namespace='default')
print(f"Scaling cycle complete: {actions_taken} actions applied")
```

#### Example 8: Continuous Monitoring
```python
# Run continuous monitoring for 3 cycles
autoscaler.run_continuous_monitoring(
    interval_seconds=60,    # Check every 60 seconds
    namespace='default',    # Monitor default namespace
    max_cycles=3           # Run 3 cycles then stop
)

# For infinite monitoring, omit max_cycles:
# autoscaler.run_continuous_monitoring(interval_seconds=60, namespace='default')
# Press Ctrl+C to stop
```--

#### Example 9: Get Scaling Report
```python
# Generate comprehensive scaling report
report = autoscaler.get_scaling_report()

print(f"Total Scaling Events: {report['total_scaling_events']}")
print(f"Scale Up Events: {report['scale_up_events']}")
print(f"Scale Down Events: {report['scale_down_events']}")
print(f"Deployments Scaled: {report['deployments_scaled']}")

print("\nRecent Events:")
for event in report['recent
