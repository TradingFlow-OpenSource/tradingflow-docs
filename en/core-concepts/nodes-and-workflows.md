# Nodes and Workflows

TradingFlow is a workflow application. So what are workflows and the smallest unit within them—nodes?

## Definition and Functionality of Nodes

First, let's explain the details of "nodes." A node is a functional module that encapsulates complex algorithms, data sources, or logical operations. For example, an X Listener node only requires users to fill in the X account they want to monitor, and it can retrieve the latest tweets from that specified account during runtime.

A node has "inputs" and "outputs". In the example just mentioned, the account filled in (such as @realDonaldTrump) is the input, and Trump's tweets are the output. Through this input-output mechanism, nodes can receive data, process data, and pass results to other nodes.

## Data Flow and Signal Transmission

When multiple nodes' inputs and outputs are connected, data can be transmitted through the workflow composed of these nodes. We call the transmitted data content "signals" and the process output during a single run "logs".

This design allows complex business logic to be decomposed into multiple simple functional modules, with each module focusing on specific tasks. Nodes communicate through signals, forming a complete data processing chain. The logging system records each node's execution status and output results, facilitating debugging and monitoring.

## Node Components

In summary, a node consists of four parts:

- **Inputs** - Receive external data or information passed from upstream nodes
- **Outputs** - Send processed results to downstream nodes
- **Signals** - The actual data content flowing between nodes
- **Logs** - Detailed records of node execution process and results

Multiple properly connected nodes form a workflow. Workflows can be simple linear processes or complex logic involving branches, loops, and conditional judgments.

## Building Workflows

Through TradingFlow's visual interface, users can assemble nodes by dragging and dropping, and connect their input and output ports with clicks. The platform provides a rich variety of node types, covering data collection, analysis processing, conditional judgment, trade execution, and other aspects, enabling users to quickly build automated strategies that meet their needs.

---

**Next Step:** Check out [Node Details](../node-details/index.md) to explore all available node types and functionalities.
