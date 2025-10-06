# AI Model Node

AI Model Node is an intelligent computation node that calls Large Language Models (like GPT-4) to analyze data, generate trading insights, and make intelligent decisions.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `ai_model_node` |
| **Display Name** | AI Model |
| **Category** | Compute (AI Processing) |
| **Icon** | 🤖 Robot Icon |
| **Handle Color** | Sky (Light Blue) |

---

## Functionality

AI Model Node receives various data signals from upstream nodes as context, processes them through Large Language Models, and generates structured response data. The node has intelligent output formatting capabilities that automatically adjust output format based on downstream node requirements.

**Main Uses:**
- Analyze market data and generate trading recommendations
- Interpret social media content to extract trading signals
- Make intelligent decisions based on multi-source data
- Generate structured data in specific formats
- Automate complex data analysis tasks

**Core Features:**
- 🎯 **Smart Formatting**: Automatically generates JSON format based on downstream requirements
- 🔗 **Multi-Signal Fusion**: Integrates multiple input signals for comprehensive analysis
- 📊 **Structured Extraction**: Automatically extracts structured data from AI responses
- 🤖 **Multi-Model Support**: Supports GPT-3.5, GPT-4, and more models

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/ai-model-node.md) - Complete documentation (~14,000 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/compute/AIModelNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/ai_model_node.py`

---

**Maintained by:** TradingFlow Development Team  
**Version:** 1.0.0
