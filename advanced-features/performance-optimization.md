# Performance Optimization

Optimize your TradingFlow strategies and infrastructure for maximum performance and efficiency.

## Strategy Optimization

### Backtesting Best Practices

#### Efficient Backtesting Framework

```python
import pandas as pd
import numpy as np
from typing import Dict, List, Tuple
import time

class OptimizedBacktester:
    def __init__(self, data: pd.DataFrame, initial_capital: float = 10000):
        self.data = data
        self.initial_capital = initial_capital
        self.reset()

    def reset(self):
        self.capital = self.initial_capital
        self.positions = {}
        self.trades = []
        self.portfolio_values = []

    def vectorized_backtest(self, signals: pd.Series) -> Dict[str, float]:
        """Vectorized backtesting for better performance"""
        # Align signals with price data
        aligned_signals = signals.reindex(self.data.index).fillna(0)

        # Calculate position changes
        position_changes = aligned_signals.diff().fillna(0)

        # Calculate returns
        price_returns = self.data['close'].pct_change().fillna(0)

        # Calculate strategy returns
        strategy_returns = aligned_signals.shift(1) * price_returns

        # Calculate cumulative returns
        cumulative_returns = (1 + strategy_returns).cumprod()

        # Calculate portfolio values
        portfolio_values = self.initial_capital * cumulative_returns

        # Calculate performance metrics
        total_return = portfolio_values.iloc[-1] / self.initial_capital - 1
        volatility = strategy_returns.std() * np.sqrt(252)  # Annualized
        sharpe_ratio = total_return / volatility if volatility > 0 else 0

        # Calculate drawdown
        peak = portfolio_values.expanding().max()
        drawdown = (portfolio_values - peak) / peak
        max_drawdown = drawdown.min()

        return {
            'total_return': total_return,
            'volatility': volatility,
            'sharpe_ratio': sharpe_ratio,
            'max_drawdown': max_drawdown,
            'final_portfolio_value': portfolio_values.iloc[-1]
        }

    def walk_forward_optimization(self, strategy_func, window_size: int = 252) -> List[Dict]:
        """Walk-forward analysis for robust optimization"""
        results = []

        for i in range(window_size, len(self.data), 21):  # Monthly retraining
            # Training data
            train_data = self.data.iloc[i-window_size:i]

            # Test data (next month)
            test_start = i
            test_end = min(i + 21, len(self.data))
            test_data = self.data.iloc[test_start:test_end]

            if len(test_data) < 10:  # Skip if insufficient test data
                continue

            # Optimize parameters on training data
            optimal_params = self.optimize_parameters(strategy_func, train_data)

            # Test on out-of-sample data
            backtester = OptimizedBacktester(test_data, self.initial_capital)
            signals = strategy_func(test_data, **optimal_params)
            performance = backtester.vectorized_backtest(signals)

            results.append({
                'period': test_data.index[0],
                'parameters': optimal_params,
                'performance': performance,
                'in_sample_return': self.calculate_return(train_data),
                'out_sample_return': performance['total_return']
            })

        return results

    def optimize_parameters(self, strategy_func, data: pd.DataFrame) -> Dict:
        """Parameter optimization with early stopping"""
        # Define parameter ranges
        param_ranges = {
            'fast_period': range(5, 21, 2),
            'slow_period': range(20, 51, 5),
            'rsi_period': range(10, 21, 2)
        }

        best_params = {}
        best_sharpe = -np.inf

        # Grid search with early stopping
        for fast_period in param_ranges['fast_period']:
            for slow_period in param_ranges['slow_period']:
                if slow_period <= fast_period:
                    continue

                for rsi_period in param_ranges['rsi_period']:
                    params = {
                        'fast_period': fast_period,
                        'slow_period': slow_period,
                        'rsi_period': rsi_period
                    }

                    signals = strategy_func(data, **params)
                    performance = self.vectorized_backtest(signals)

                    if performance['sharpe_ratio'] > best_sharpe:
                        best_sharpe = performance['sharpe_ratio']
                        best_params = params

        return best_params

    def calculate_return(self, data: pd.DataFrame) -> float:
        """Calculate simple return for data period"""
        if len(data) < 2:
            return 0.0

        initial_price = data['close'].iloc[0]
        final_price = data['close'].iloc[-1]

        return (final_price - initial_price) / initial_price
```

### Memory-Efficient Processing

#### Chunked Data Processing

```python
class MemoryEfficientProcessor:
    def __init__(self, chunk_size: int = 10000):
        self.chunk_size = chunk_size

    def process_large_dataset(self, data_generator, processor_func):
        """Process large datasets in chunks to manage memory"""
        results = []

        chunk = []
        for item in data_generator:
            chunk.append(item)

            if len(chunk) >= self.chunk_size:
                processed_chunk = processor_func(chunk)
                results.extend(processed_chunk)
                chunk = []  # Clear chunk to free memory

        # Process remaining items
        if chunk:
            processed_chunk = processor_func(chunk)
            results.extend(processed_chunk)

        return results

    def calculate_rolling_indicators(self, data: pd.DataFrame, window: int) -> pd.DataFrame:
        """Memory-efficient rolling calculations"""
        result = pd.DataFrame(index=data.index)

        # Process in chunks to avoid loading full dataset
        for i in range(window, len(data), self.chunk_size):
            start_idx = max(0, i - window)
            end_idx = min(len(data), i + self.chunk_size)

            chunk_data = data.iloc[start_idx:end_idx]

            # Calculate indicators for this chunk
            result.loc[chunk_data.index, 'sma'] = chunk_data['close'].rolling(window=window).mean()
            result.loc[chunk_data.index, 'rsi'] = self._calculate_rsi_chunk(chunk_data, window)

        return result

    def _calculate_rsi_chunk(self, data: pd.DataFrame, period: int) -> pd.Series:
        """Calculate RSI for a data chunk"""
        delta = data['close'].diff()
        gain = (delta.where(delta > 0, 0)).rolling(window=period).mean()
        loss = (-delta.where(delta < 0, 0)).rolling(window=period).mean()
        rs = gain / loss
        rsi = 100 - (100 / (1 + rs))
        return rsi
```

## Infrastructure Optimization

### Database Optimization

#### Query Optimization

```sql
-- Optimized queries for high-frequency trading data

-- 1. Partitioned tables for time-series data
CREATE TABLE trades (
    id BIGSERIAL,
    symbol VARCHAR(10),
    price DECIMAL(20,8),
    quantity DECIMAL(20,8),
    timestamp TIMESTAMPTZ,
    exchange VARCHAR(20)
) PARTITION BY RANGE (timestamp);

-- Create monthly partitions
CREATE TABLE trades_2024_01 PARTITION OF trades
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- 2. Optimized indexes for common queries
CREATE INDEX CONCURRENTLY idx_trades_symbol_timestamp
ON trades (symbol, timestamp DESC);

CREATE INDEX CONCURRENTLY idx_trades_timestamp_symbol
ON trades (timestamp DESC, symbol);

-- 3. Materialized views for aggregated data
CREATE MATERIALIZED VIEW hourly_volume AS
SELECT
    symbol,
    date_trunc('hour', timestamp) as hour,
    SUM(quantity) as total_volume,
    AVG(price) as avg_price,
    MIN(price) as min_price,
    MAX(price) as max_price
FROM trades
WHERE timestamp >= CURRENT_TIMESTAMP - INTERVAL '30 days'
GROUP BY symbol, date_trunc('hour', timestamp)
WITH NO DATA;

-- 4. Efficient pagination
CREATE OR REPLACE FUNCTION get_trades_paginated(
    p_symbol VARCHAR(10),
    p_limit INTEGER DEFAULT 100,
    p_offset INTEGER DEFAULT 0
)
RETURNS TABLE (
    id BIGINT,
    symbol VARCHAR(10),
    price DECIMAL(20,8),
    quantity DECIMAL(20,8),
    timestamp TIMESTAMPTZ
)
LANGUAGE SQL
STABLE
AS $$
    SELECT id, symbol, price, quantity, timestamp
    FROM trades
    WHERE symbol = p_symbol
    ORDER BY timestamp DESC
    LIMIT p_limit
    OFFSET p_offset;
$$;
```

#### Connection Pooling

```python
import asyncpg
from typing import Optional, List, Dict, Any

class OptimizedConnectionPool:
    def __init__(self, dsn: str, min_size: int = 10, max_size: int = 20):
        self.dsn = dsn
        self.min_size = min_size
        self.max_size = max_size
        self.pool: Optional[asyncpg.Pool] = None

    async def initialize(self):
        """Initialize connection pool"""
        self.pool = await asyncpg.create_pool(
            self.dsn,
            min_size=self.min_size,
            max_size=self.max_size,
            command_timeout=60,
            max_inactive_connection_lifetime=300
        )

    async def execute_query(self, query: str, params: tuple = None) -> List[Dict[str, Any]]:
        """Execute query using connection pool"""
        async with self.pool.acquire() as connection:
            if params:
                result = await connection.fetch(query, *params)
            else:
                result = await connection.fetch(query)

            return [dict(row) for row in result]

    async def execute_batch(self, queries: List[Dict[str, str]]) -> List[Any]:
        """Execute multiple queries in batch"""
        async with self.pool.acquire() as connection:
            async with connection.transaction():
                results = []
                for query_info in queries:
                    if 'params' in query_info:
                        result = await connection.fetch(
                            query_info['query'],
                            *query_info['params']
                        )
                    else:
                        result = await connection.fetch(query_info['query'])
                    results.append(result)
                return results

    async def close(self):
        """Close connection pool"""
        if self.pool:
            await self.pool.close()

    async def get_pool_stats(self) -> Dict[str, Any]:
        """Get connection pool statistics"""
        if not self.pool:
            return {}

        return {
            'min_size': self.pool._minsize,
            'max_size': self.pool._maxsize,
            'size': len(self.pool._holders),
            'free': self.pool._queue.qsize()
        }
```

### Caching Strategies

#### Multi-Level Caching

```python
import redis
import asyncio
from typing import Any, Optional, Dict
import json
import hashlib

class MultiLevelCache:
    def __init__(self, redis_url: str = "redis://localhost:6379"):
        self.redis = redis.from_url(redis_url)
        self.l1_cache: Dict[str, Dict[str, Any]] = {}
        self.l1_ttl = 300  # 5 minutes

    def _get_cache_key(self, key: str, params: Dict = None) -> str:
        """Generate cache key with parameter hashing"""
        if params:
            param_str = json.dumps(params, sort_keys=True)
            param_hash = hashlib.md5(param_str.encode()).hexdigest()[:8]
            return f"{key}:{param_hash}"
        return key

    async def get(self, key: str, params: Dict = None) -> Optional[Any]:
        """Get value from multi-level cache"""
        cache_key = self._get_cache_key(key, params)

        # Check L1 cache first
        if cache_key in self.l1_cache:
            entry = self.l1_cache[cache_key]
            if asyncio.get_event_loop().time() - entry['timestamp'] < self.l1_ttl:
                return entry['value']
            else:
                del self.l1_cache[cache_key]

        # Check Redis L2 cache
        try:
            data = self.redis.get(cache_key)
            if data:
                value = json.loads(data)
                # Populate L1 cache
                self.l1_cache[cache_key] = {
                    'value': value,
                    'timestamp': asyncio.get_event_loop().time()
                }
                return value
        except redis.RedisError:
            pass  # Redis unavailable, continue

        return None

    async def set(self, key: str, value: Any, ttl: int = 3600, params: Dict = None):
        """Set value in multi-level cache"""
        cache_key = self._get_cache_key(key, params)

        # Set L1 cache
        self.l1_cache[cache_key] = {
            'value': value,
            'timestamp': asyncio.get_event_loop().time()
        }

        # Set Redis L2 cache
        try:
            self.redis.setex(cache_key, ttl, json.dumps(value))
        except redis.RedisError:
            pass  # Redis unavailable, skip

    async def invalidate_pattern(self, pattern: str):
        """Invalidate cache keys matching pattern"""
        try:
            keys = self.redis.keys(pattern)
            if keys:
                self.redis.delete(*keys)

            # Also clear L1 cache
            keys_to_delete = [k for k in self.l1_cache.keys() if pattern.replace('*', '') in k]
            for key in keys_to_delete:
                del self.l1_cache[key]
        except redis.RedisError:
            pass

    async def get_cache_stats(self) -> Dict[str, Any]:
        """Get cache performance statistics"""
        try:
            redis_info = self.redis.info()
            return {
                'l1_entries': len(self.l1_cache),
                'redis_connected_clients': redis_info.get('connected_clients', 0),
                'redis_used_memory': redis_info.get('used_memory_human', '0B'),
                'redis_hit_rate': redis_info.get('keyspace_hits', 0) /
                                max(redis_info.get('keyspace_hits', 0) +
                                    redis_info.get('keyspace_misses', 0), 1)
            }
        except redis.RedisError:
            return {'redis_status': 'disconnected'}
```

### Message Queue Optimization

#### Asynchronous Processing

```python
import asyncio
import aio_pika
from typing import Callable, Any, Dict
import json

class OptimizedMessageQueue:
    def __init__(self, amqp_url: str):
        self.amqp_url = amqp_url
        self.connection = None
        self.channel = None
        self.consumers = {}

    async def connect(self):
        """Establish connection to message broker"""
        self.connection = await aio_pika.connect_robust(self.amqp_url)
        self.channel = await self.connection.channel()

        # Set QoS for fair dispatching
        await self.channel.set_qos(prefetch_count=10)

    async def publish_message(self, exchange: str, routing_key: str, message: Dict[str, Any]):
        """Publish message with optimization"""
        await self.channel.default_exchange.publish(
            aio_pika.Message(
                body=json.dumps(message).encode(),
                delivery_mode=aio_pika.DeliveryMode.PERSISTENT,  # Persistent messages
                headers={'timestamp': asyncio.get_event_loop().time()}
            ),
            routing_key=routing_key
        )

    async def consume_messages(self, queue_name: str, callback: Callable, concurrency: int = 5):
        """Consume messages with controlled concurrency"""
        queue = await self.channel.declare_queue(queue_name, durable=True)

        semaphore = asyncio.Semaphore(concurrency)

        async def process_message(message: aio_pika.IncomingMessage):
            async with semaphore:
                try:
                    data = json.loads(message.body.decode())
                    await callback(data)
                    await message.ack()
                except Exception as e:
                    print(f"Error processing message: {e}")
                    await message.reject(requeue=True)

        await queue.consume(process_message)

        # Store consumer for cleanup
        self.consumers[queue_name] = queue

    async def batch_publish(self, messages: List[Dict[str, Any]], exchange: str, routing_key: str):
        """Batch publish multiple messages efficiently"""
        async with self.channel.transaction():
            for message in messages:
                await self.publish_message(exchange, routing_key, message)

    async def create_delayed_queue(self, queue_name: str, delay_seconds: int):
        """Create queue with message delay"""
        # Create main queue
        main_queue = await self.channel.declare_queue(f"{queue_name}_main", durable=True)

        # Create delay queue
        delay_queue = await self.channel.declare_queue(f"{queue_name}_delay", durable=True)

        # Create delay exchange
        delay_exchange = await self.channel.declare_exchange(
            f"{queue_name}_delay_exchange",
            aio_pika.ExchangeType.DIRECT,
            durable=True
        )

        # Bind queues
        await delay_queue.bind(delay_exchange, f"{queue_name}_delay")
        await main_queue.bind(delay_exchange, f"{queue_name}_main")

        # Set message TTL and dead letter exchange
        await self.channel.queue_bind(
            delay_queue,
            delay_exchange,
            f"{queue_name}_delay",
            arguments={
                'x-message-ttl': delay_seconds * 1000,
                'x-dead-letter-exchange': delay_exchange.name,
                'x-dead-letter-routing-key': f"{queue_name}_main"
            }
        )

    async def close(self):
        """Close connections"""
        if self.channel:
            await self.channel.close()
        if self.connection:
            await self.connection.close()
```

## Algorithm Optimization

### Vectorized Calculations

```python
import numpy as np
import pandas as pd
from numba import jit, njit
import vectorbt as vbt

class VectorizedCalculations:
    @staticmethod
    @njit
    def calculate_rsi_numba(prices: np.ndarray, period: int) -> np.ndarray:
        """Numba-optimized RSI calculation"""
        length = len(prices)
        rsi = np.full(length, np.nan)

        if length < period + 1:
            return rsi

        gains = np.zeros(length)
        losses = np.zeros(length)

        # Calculate price changes
        for i in range(1, length):
            change = prices[i] - prices[i-1]
            if change > 0:
                gains[i] = change
            else:
                losses[i] = -change

        # Calculate initial averages
        avg_gain = np.mean(gains[1:period+1])
        avg_loss = np.mean(losses[1:period+1])

        if period < length:
            rsi[period] = 100 - (100 / (1 + avg_gain / max(avg_loss, 1e-10)))

        # Calculate subsequent values
        for i in range(period + 1, length):
            avg_gain = (avg_gain * (period - 1) + gains[i]) / period
            avg_loss = (avg_loss * (period - 1) + losses[i]) / period
            rsi[i] = 100 - (100 / (1 + avg_gain / max(avg_loss, 1e-10)))

        return rsi

    @staticmethod
    def vectorbt_portfolio_optimization(self, price_data: pd.DataFrame, strategies: List[str]):
        """VectorBT-based portfolio optimization"""
        # Create portfolio from price data
        pf = vbt.Portfolio.from_price_data(price_data)

        # Run multiple strategies in parallel
        results = {}
        for strategy in strategies:
            if strategy == 'buy_and_hold':
                result = pf.total_return()
            elif strategy == 'moving_average':
                fast_ma = vbt.MA.run(price_data, window=20)
                slow_ma = vbt.MA.run(price_data, window=50)
                entries = fast_ma.ma_above(slow_ma.ma)
                exits = fast_ma.ma_below(slow_ma.ma)
                result = pf.total_return(entries=entries, exits=exits)
            elif strategy == 'rsi':
                rsi = vbt.RSI.run(price_data).rsi
                entries = rsi.rsi_below(30)
                exits = rsi.rsi_above(70)
                result = pf.total_return(entries=entries, exits=exits)

            results[strategy] = result

        return results

    @staticmethod
    def pandas_vectorized_indicators(data: pd.DataFrame) -> pd.DataFrame:
        """Pandas vectorized technical indicators"""
        result = pd.DataFrame(index=data.index)

        # Moving averages
        result['sma_20'] = data['close'].rolling(window=20).mean()
        result['ema_12'] = data['close'].ewm(span=12).mean()
        result['ema_26'] = data['close'].ewm(span=26).mean()

        # MACD
        result['macd'] = result['ema_12'] - result['ema_26']
        result['signal'] = result['macd'].ewm(span=9).mean()
        result['histogram'] = result['macd'] - result['signal']

        # Bollinger Bands
        sma_20 = result['sma_20']
        std_20 = data['close'].rolling(window=20).std()
        result['bb_upper'] = sma_20 + (std_20 * 2)
        result['bb_lower'] = sma_20 - (std_20 * 2)
        result['bb_middle'] = sma_20

        # RSI
        delta = data['close'].diff()
        gain = (delta.where(delta > 0, 0)).rolling(window=14).mean()
        loss = (-delta.where(delta < 0, 0)).rolling(window=14).mean()
        rs = gain / loss
        result['rsi'] = 100 - (100 / (1 + rs))

        # Stochastic Oscillator
        lowest_low = data['low'].rolling(window=14).min()
        highest_high = data['high'].rolling(window=14).max()
        result['stoch_k'] = 100 * ((data['close'] - lowest_low) / (highest_high - lowest_low))
        result['stoch_d'] = result['stoch_k'].rolling(window=3).mean()

        return result
```

### Parallel Processing

```python
import multiprocessing as mp
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor
import asyncio
from typing import List, Dict, Any

class ParallelProcessor:
    def __init__(self, max_workers: int = None):
        self.max_workers = max_workers or mp.cpu_count()

    def parallel_backtest(self, strategies: List[Dict[str, Any]], data: pd.DataFrame) -> List[Dict]:
        """Parallel backtesting of multiple strategies"""
        with ProcessPoolExecutor(max_workers=self.max_workers) as executor:
            futures = [
                executor.submit(self._backtest_single_strategy, strategy, data)
                for strategy in strategies
            ]

            results = []
            for future in futures:
                try:
                    result = future.result(timeout=300)  # 5 minute timeout
                    results.append(result)
                except Exception as e:
                    results.append({'error': str(e), 'strategy': strategy.get('name', 'unknown')})

            return results

    def _backtest_single_strategy(self, strategy: Dict[str, Any], data: pd.DataFrame) -> Dict:
        """Backtest single strategy (runs in separate process)"""
        try:
            backtester = OptimizedBacktester(data)
            signals = self._generate_signals(strategy, data)
            performance = backtester.vectorized_backtest(signals)

            return {
                'strategy_name': strategy.get('name', 'unnamed'),
                'parameters': strategy.get('params', {}),
                'performance': performance,
                'success': True
            }
        except Exception as e:
            return {
                'strategy_name': strategy.get('name', 'unnamed'),
                'error': str(e),
                'success': False
            }

    async def async_data_fetching(self, symbols: List[str]) -> Dict[str, pd.DataFrame]:
        """Asynchronous data fetching for multiple symbols"""
        async def fetch_symbol_data(symbol: str) -> tuple[str, pd.DataFrame]:
            # Simulate API call
            await asyncio.sleep(0.1)  # API latency simulation
            data = await self._fetch_symbol_data(symbol)
            return symbol, data

        tasks = [fetch_symbol_data(symbol) for symbol in symbols]
        results = await asyncio.gather(*tasks, return_exceptions=True)

        data_dict = {}
        for result in results:
            if isinstance(result, Exception):
                continue  # Handle exceptions as needed
            symbol, data = result
            data_dict[symbol] = data

        return data_dict

    def parallel_indicator_calculation(self, data: pd.DataFrame, indicators: List[str]) -> pd.DataFrame:
        """Parallel calculation of multiple indicators"""
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            futures = {}
            for indicator in indicators:
                if indicator == 'rsi':
                    future = executor.submit(self._calculate_rsi, data)
                elif indicator == 'macd':
                    future = executor.submit(self._calculate_macd, data)
                elif indicator == 'bollinger':
                    future = executor.submit(self._calculate_bollinger, data)
                # Add more indicators as needed

                futures[indicator] = future

            results = {}
            for indicator, future in futures.items():
                try:
                    results[indicator] = future.result(timeout=60)
                except Exception as e:
                    results[indicator] = pd.Series(index=data.index, dtype=float)

            return pd.DataFrame(results)

    def _calculate_rsi(self, data: pd.DataFrame) -> pd.Series:
        """Calculate RSI indicator"""
        delta = data['close'].diff()
        gain = (delta.where(delta > 0, 0)).rolling(window=14).mean()
        loss = (-delta.where(delta < 0, 0)).rolling(window=14).mean()
        rs = gain / loss
        return 100 - (100 / (1 + rs))

    def _calculate_macd(self, data: pd.DataFrame) -> pd.DataFrame:
        """Calculate MACD indicator"""
        ema12 = data['close'].ewm(span=12).mean()
        ema26 = data['close'].ewm(span=26).mean()
        macd = ema12 - ema26
        signal = macd.ewm(span=9).mean()
        histogram = macd - signal

        return pd.DataFrame({
            'macd': macd,
            'signal': signal,
            'histogram': histogram
        })

    def _calculate_bollinger(self, data: pd.DataFrame) -> pd.DataFrame:
        """Calculate Bollinger Bands"""
        sma = data['close'].rolling(window=20).mean()
        std = data['close'].rolling(window=20).std()

        return pd.DataFrame({
            'upper': sma + (std * 2),
            'middle': sma,
            'lower': sma - (std * 2)
        })
```

---

Performance optimization is crucial for scalable trading systems. These techniques help manage computational resources efficiently while maintaining fast execution and low latency for your trading strategies.

## Next Steps
- [Monitoring & Analytics →](monitoring-analytics.md) - Performance monitoring
- [Integration Ecosystem →](integration-ecosystem.md) - Optimize external integrations
- [Deployment & Operations →](../deployment-operations/) - Production deployment optimization
