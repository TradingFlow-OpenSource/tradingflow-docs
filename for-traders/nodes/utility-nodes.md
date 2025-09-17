# Utility Nodes

Utility nodes provide supporting infrastructure, data transformation, timing controls, and workflow management capabilities.

## Timer Node

**Purpose**: Schedule and trigger workflow execution at specified intervals
**Node Type**: `timer_node`
**Category**: Utility

#### Configuration Parameters
```javascript
{
  schedule_type: "recurring",       // recurring, one_time, cron
  frequency: "daily",              // hourly, daily, weekly, monthly
  time: "09:00",                   // Execution time (HH:MM format)
  timezone: "UTC",                 // Timezone for scheduling
  start_date: "2024-01-15",        // Optional start date
  end_date: null,                  // Optional end date
  max_executions: 0,               // 0 = unlimited
  enabled: true                    // Enable/disable timer
}
```

#### Advanced Scheduling Options

**Cron Expression**
```javascript
{
  schedule_type: "cron",
  cron_expression: "0 9 * * 1",    // Every Monday at 9 AM
  timezone: "America/New_York"
}
```

**Custom Intervals**
```javascript
{
  schedule_type: "interval",
  interval_seconds: 3600,          // Every hour
  start_immediately: false,
  max_runtime: 86400               // Stop after 24 hours
}
```

#### Output Handles
- `trigger_signal` (object): Timer execution signal
  ```javascript
  {
    execution_id: "timer_exec_123456",
    scheduled_time: "2024-01-15T09:00:00Z",
    actual_time: "2024-01-15T09:00:02Z",
    next_execution: "2024-01-16T09:00:00Z",
    execution_count: 45,
    status: "completed"
  }
  ```

#### Common Use Cases

- **DCA Strategies**: Daily token purchases
- **Rebalancing**: Weekly portfolio adjustments
- **Report Generation**: End-of-day performance reports
- **Market Scanning**: Hourly market analysis

## Data Transformation Node

**Purpose**: Process, filter, and transform data between nodes
**Node Type**: `transform_node`
**Category**: Utility

#### Configuration Parameters
```javascript
{
  transformation_type: "filter",    // filter, map, aggregate, normalize
  input_format: "json",            // json, csv, array, object
  output_format: "json",           // json, csv, array, object
  validation_rules: {
    required_fields: ["price", "volume"],
    data_types: {
      price: "number",
      volume: "number"
    }
  }
}
```

#### Transformation Types

**Filter Data**
```javascript
{
  transformation_type: "filter",
  conditions: [
    {
      field: "volume",
      operator: "greater_than",
      value: 1000000               // Filter trades > $1M volume
    },
    {
      field: "price_change",
      operator: "between",
      min: -0.05,
      max: 0.05                   // Price change between -5% and +5%
    }
  ]
}
```

**Map Fields**
```javascript
{
  transformation_type: "map",
  field_mappings: {
    "open_price": "price_open",
    "close_price": "price_close",
    "trading_volume": "volume",
    "timestamp": "time"
  },
  data_type_conversions: {
    "volume": "number",
    "time": "timestamp"
  }
}
```

**Aggregate Data**
```javascript
{
  transformation_type: "aggregate",
  group_by: "symbol",             // Group by trading pair
  aggregations: {
    "price": "average",           // Average price
    "volume": "sum",              // Total volume
    "high": "max",                // Highest price
    "low": "min"                  // Lowest price
  },
  time_window: "1h"               // Aggregate over 1-hour windows
}
```

#### Input Handles
- `input_data` (any): Raw data to transform
- `transformation_rules` (object): Dynamic transformation parameters

#### Output Handles
- `transformed_data` (any): Processed data
- `transformation_stats` (object): Processing statistics
  ```javascript
  {
    records_processed: 1250,
    records_filtered: 234,
    records_output: 1016,
    processing_time: 0.15,        // seconds
    errors: 0,
    data_quality_score: 0.98
  }
  ```

## Math Operation Node

**Purpose**: Perform mathematical calculations and computations
**Node Type**: `math_node`
**Category**: Utility

#### Configuration Parameters
```javascript
{
  operation: "add",               // add, subtract, multiply, divide, power, sqrt, log
  precision: 8,                   // Decimal precision
  error_handling: "zero_division", // How to handle errors
  input_validation: true
}
```

#### Supported Operations

| Operation | Description | Example |
|-----------|-------------|---------|
| **Basic Math** | Addition, subtraction, multiplication, division | `a + b`, `a * b` |
| **Power/Sqrt** | Exponentiation, square root | `a^b`, `√a` |
| **Logarithms** | Natural log, base-10 log | `ln(a)`, `log₁₀(a)` |
| **Trigonometry** | Sin, cos, tan functions | `sin(a)`, `cos(a)` |
| **Statistics** | Mean, median, standard deviation | `avg(array)`, `std(array)` |
| **Financial** | Percentage change, returns | `pct_change(current, previous)` |

#### Advanced Configuration

**Multi-Input Operations**
```javascript
{
  operation: "weighted_average",
  inputs: [
    { source: "price_feed_1", weight: 0.6 },
    { source: "price_feed_2", weight: 0.4 }
  ],
  normalization: true
}
```

**Time Series Operations**
```javascript
{
  operation: "moving_average",
  window_size: 20,                // 20-period moving average
  window_type: "simple",          // simple, exponential, weighted
  input_field: "close_price"
}
```

#### Input Handles
- `input_a` (number): First operand
- `input_b` (number): Second operand (for binary operations)
- `input_array` (array): Array for statistical operations

#### Output Handles
- `result` (number): Calculation result
- `calculation_details` (object): Detailed calculation breakdown
  ```javascript
  {
    operation: "moving_average",
    input_count: 20,
    result: 43250.75,
    method: "simple",
    standard_deviation: 1250.50,
    confidence_interval: [42000, 44500]
  }
  ```

## Data Storage Node

**Purpose**: Store and retrieve data from persistent storage
**Node Type**: `storage_node`
**Category**: Utility

#### Configuration Parameters
```javascript
{
  storage_type: "local",          // local, cloud, database
  data_format: "json",            // json, csv, binary
  retention_policy: {
    max_age_days: 365,            // Keep data for 1 year
    max_size_mb: 100,             // Maximum 100MB storage
    compression: true             // Enable compression
  },
  backup_enabled: true,
  encryption: "aes256"            // Data encryption
}
```

#### Storage Operations

**Store Data**
```javascript
{
  operation: "store",
  key: "btc_price_history_2024",
  data: priceDataArray,
  metadata: {
    source: "binance_api",
    interval: "1h",
    last_updated: "2024-01-15T10:30:00Z"
  }
}
```

**Retrieve Data**
```javascript
{
  operation: "retrieve",
  key: "btc_price_history_2024",
  query: {
    date_range: {
      start: "2024-01-01",
      end: "2024-01-15"
    },
    filter: {
      volume: { min: 1000000 }    // Only high volume data
    }
  }
}
```

**Update Data**
```javascript
{
  operation: "update",
  key: "portfolio_snapshot",
  data: currentPortfolio,
  merge_strategy: "replace"       // replace, merge, append
}
```

#### Input Handles
- `data` (any): Data to store or query parameters
- `operation` (string): store, retrieve, update, delete

#### Output Handles
- `stored_data` (any): Retrieved or stored data
- `storage_status` (object): Operation status
  ```javascript
  {
    operation: "store",
    key: "btc_price_history_2024",
    bytes_stored: 2048576,
    compression_ratio: 0.75,
    encryption_status: "encrypted",
    backup_status: "scheduled",
    timestamp: "2024-01-15T10:30:00Z"
  }
  ```

## Error Handler Node

**Purpose**: Manage errors and exceptions in workflow execution
**Node Type**: `error_handler_node`
**Category**: Utility

#### Configuration Parameters
```javascript
{
  error_types: ["network", "api", "calculation", "timeout"],
  retry_policy: {
    max_retries: 3,
    backoff_strategy: "exponential",  // fixed, linear, exponential
    base_delay: 1,                  // Base delay in seconds
    max_delay: 60                   // Maximum delay
  },
  fallback_actions: {
    on_failure: "use_cached_data",   // skip, retry, fallback, stop
    notification: true,
    log_level: "warning"
  }
}
```

#### Error Handling Strategies

**Retry with Backoff**
```javascript
{
  retry_policy: {
    max_retries: 5,
    backoff_strategy: "exponential",
    base_delay: 2,
    jitter: true                    // Add randomness to prevent thundering herd
  }
}
```

**Fallback Actions**
```javascript
{
  fallback_actions: {
    network_error: {
      action: "use_backup_api",
      backup_endpoints: ["api2.example.com", "api3.example.com"]
    },
    data_error: {
      action: "use_default_values",
      defaults: {
        price: "last_known_price",
        volume: 0
      }
    }
  }
}
```

#### Input Handles
- `error_input` (object): Error details from failed node
- `fallback_data` (any): Alternative data for fallback scenarios

#### Output Handles
- `error_handled` (boolean): Whether error was successfully handled
- `recovery_result` (any): Recovered data or result
- `error_report` (object): Detailed error analysis
  ```javascript
  {
    error_id: "err_123456",
    error_type: "network_timeout",
    original_error: "Connection timeout after 30 seconds",
    retry_attempts: 3,
    resolution: "fallback_data_used",
    fallback_source: "cached_data",
    timestamp: "2024-01-15T10:30:00Z",
    impact_assessment: "low"
  }
  ```

## Workflow Control Node

**Purpose**: Manage complex workflow logic with branching and conditional execution
**Node Type**: `workflow_control_node`
**Category**: Utility

#### Configuration Parameters
```javascript
{
  control_type: "conditional_branch",  // conditional_branch, parallel, sequence, loop
  execution_mode: "synchronous",      // synchronous, asynchronous
  timeout: 300,                      // Maximum execution time
  error_handling: "continue",        // continue, stop, rollback
  logging_enabled: true
}
```

#### Control Types

**Conditional Branch**
```javascript
{
  control_type: "conditional_branch",
  conditions: [
    {
      condition: "portfolio.pnl > 0.05",  // If P&L > 5%
      branch: "take_profit_branch",
      priority: 1
    },
    {
      condition: "portfolio.drawdown > 0.1",  // If drawdown > 10%
      branch: "stop_loss_branch",
      priority: 2
    }
  ],
  default_branch: "hold_branch"
}
```

**Parallel Execution**
```javascript
{
  control_type: "parallel",
  branches: [
    {
      name: "technical_analysis",
      nodes: ["rsi_calc", "macd_calc", "bollinger_calc"],
      timeout: 60
    },
    {
      name: "sentiment_analysis",
      nodes: ["twitter_feed", "news_feed"],
      timeout: 120
    }
  ],
  aggregation_method: "consensus"    // consensus, majority, first_completed
}
```

**Loop Control**
```javascript
{
  control_type: "loop",
  loop_condition: "attempts < max_retries",
  max_iterations: 5,
  loop_variables: {
    attempts: 0,
    max_retries: 3
  },
  exit_condition: "success || final_attempt"
}
```

#### Input Handles
- `control_input` (any): Data for decision making
- `branch_conditions` (object): Conditions for branching

#### Output Handles
- `control_output` (any): Result from executed branch
- `execution_path` (array): Path taken through workflow
- `control_stats` (object): Execution statistics
  ```javascript
  {
    control_type: "conditional_branch",
    branch_taken: "take_profit_branch",
    execution_time: 0.25,
    conditions_evaluated: 3,
    path: ["entry", "condition_check", "take_profit_branch", "exit"],
    performance_metrics: {
      cpu_usage: 15,
      memory_usage: 45,
      network_calls: 2
    }
  }
  ```

## Logger Node

**Purpose**: Record workflow execution data and performance metrics
**Node Type**: `logger_node`
**Category**: Utility

#### Configuration Parameters
```javascript
{
  log_level: "info",               // debug, info, warning, error
  log_destinations: ["file", "database", "cloud"],
  retention_days: 90,              // Keep logs for 90 days
  structured_logging: true,        // JSON format logs
  performance_tracking: true,      // Track execution metrics
  error_tracking: true             // Special handling for errors
}
```

#### Log Categories

**Execution Logs**
```javascript
{
  category: "execution",
  level: "info",
  message: "Strategy execution completed",
  data: {
    strategy_id: "dca_001",
    execution_time: 2.34,
    trades_executed: 1,
    status: "success"
  }
}
```

**Performance Logs**
```javascript
{
  category: "performance",
  metrics: {
    node_execution_time: 1.23,
    memory_peak: 150,
    api_calls: 5,
    data_processed: 1024
  },
  bottlenecks: ["api_call_3"],
  optimization_suggestions: ["cache_results"]
}
```

**Error Logs**
```javascript
{
  category: "error",
  level: "error",
  error_type: "network_timeout",
  stack_trace: "...",
  context: {
    node_id: "price_feed_1",
    retry_attempt: 2,
    upstream_data: "partial"
  },
  recovery_action: "fallback_to_cache"
}
```

#### Input Handles
- `log_data` (any): Data to be logged
- `log_level` (string): Log severity level
- `context` (object): Additional context information

#### Output Handles
- `log_entry` (object): Confirmed log entry
- `log_stats` (object): Logging statistics
  ```javascript
  {
    log_id: "log_789123",
    timestamp: "2024-01-15T10:30:00Z",
    level: "info",
    size_bytes: 512,
    destinations: ["file", "database"],
    retention_status: "active",
    search_tags: ["strategy", "dca", "execution"]
  }
  ```

---

Utility nodes provide the infrastructure and control mechanisms that make complex trading workflows possible. They handle timing, data processing, error management, and workflow coordination.

## Next Steps
- [Building Your First Strategy →](../first-strategy.md) - Put utility nodes to work
- [Node Reference Guide →](../node-reference-guide.md) - Complete node overview
- [Advanced Trading Patterns →](../advanced-trading-patterns.md) - Complex workflows
