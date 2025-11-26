# AutoParts Inc. AI Agents - n8n Simulation Workflow

## Simulation Overview

This document outlines the n8n workflow simulation for the AutoParts Inc. Smart Manufacturing Implementation. The simulation demonstrates how the three AI agent types (Predictive Quality Assurance, Predictive Maintenance, and Adaptive Production Planning) would interact within an n8n environment to optimize manufacturing operations.

## Workflow Architecture

### Main Workflow: "AutoParts Manufacturing Optimization"

The main workflow orchestrates three specialized sub-workflows representing each AI agent type, with data flows and decision points that simulate real manufacturing scenarios.

## Agent Workflow Specifications

### 1. Predictive Quality Assurance (PQA) Agent Workflow

**Workflow Name**: `pqa_agent_analysis`
**Purpose**: Analyze sensor data to predict quality issues before they occur

#### Node Structure:

**Trigger Node: Quality Data Ingestion**
```json
{
  "nodeType": "Webhook",
  "nodeName": "quality_data_trigger",
  "parameters": {
    "httpMethod": "POST",
    "path": "quality-sensor-data",
    "responseMode": "onReceived",
    "options": {
      "rawBody": false
    }
  }
}
```

**Processing Nodes:**

1. **Data Validation Node**
   - Type: Function
   - Purpose: Validate incoming sensor data format and completeness
   - Logic: Check for required fields (temperature, pressure, vibration, timestamp)

2. **Sensor Data Preprocessing Node**
   - Type: Function
   - Purpose: Normalize and clean sensor readings
   - Logic: Apply scaling, remove outliers, calculate moving averages

3. **Quality Prediction Node**
   - Type: AI/LLM
   - Purpose: Analyze preprocessed data for defect probability
   - Model: Custom quality prediction model (simulated)
   - Input: Sensor parameters, historical patterns
   - Output: Defect probability score, confidence level

4. **Decision Router Node**
   - Type: Switch
   - Purpose: Route based on defect probability threshold
   - Logic: If probability > 0.7 → High Priority Alert
   - Logic: If probability 0.3-0.7 → Medium Priority Alert
   - Logic: If probability < 0.3 → Normal Operation

**Alert & Action Nodes:**

5. **High Priority Alert Node**
   - Type: Email/Slack
   - Purpose: Immediate notification to quality manager
   - Content: Defect probability, recommended actions, affected production line

6. **Production Line Adjustment Node**
   - Type: HTTP Request
   - Purpose: Adjust production parameters automatically
   - API: Manufacturing execution system (MES) API
   - Actions: Reduce speed, increase inspection frequency

### 2. Predictive Maintenance (PM) Agent Workflow

**Workflow Name**: `pm_agent_monitoring`
**Purpose**: Monitor equipment health and predict maintenance needs

#### Node Structure:

**Trigger Node: Equipment Telemetry**
```json
{
  "nodeType": "Interval",
  "nodeName": "equipment_check",
  "parameters": {
    "rule": {
      "interval": [{"field": "minutes", "value": 15}]
    }
  }
}
```

**Processing Nodes:**

1. **Equipment Data Collection Node**
   - Type: HTTP Request
   - Purpose: Retrieve current equipment sensor readings
   - Sources: Vibration sensors, temperature monitors, acoustic sensors

2. **Health Assessment Node**
   - Type: Function
   - Purpose: Calculate equipment health score
   - Logic: Weighted combination of sensor parameters
   - Health indicators: Vibration trend, temperature variance, acoustic anomalies

3. **Failure Prediction Node**
   - Type: AI/LLM
   - Purpose: Predict time to failure and maintenance requirements
   - Model: Failure prediction algorithm
   - Output: Days to failure, recommended maintenance actions

4. **Maintenance Scheduler Node**
   - Type: Function
   - Purpose: Optimize maintenance scheduling
   - Logic: Consider production schedule, maintenance resources, failure urgency

**Action Nodes:**

5. **Maintenance Work Order Node**
   - Type: HTTP Request
   - Purpose: Create work order in CMMS system
   - Integration: Computerized Maintenance Management System API

6. **Inventory Check Node**
   - Type: HTTP Request
   - Purpose: Check spare parts availability
   - System: Inventory management system
   - Trigger: Automatic parts ordering if low stock

### 3. Adaptive Production Planning (APP) Agent Workflow

**Workflow Name**: `app_agent_planning`
**Purpose**: Dynamically adjust production schedules based on real-time conditions

#### Node Structure:

**Trigger Node: Order & Status Updates**
```json
{
  "nodeType": "Webhook",
  "nodeName": "production_update",
  "parameters": {
    "httpMethod": "POST",
    "path": "production-status",
    "responseMode": "onReceived"
  }
}
```

**Processing Nodes:**

1. **Production Status Analyzer**
   - Type: Function
   - Purpose: Analyze current production status and constraints
   - Inputs: Machine availability, quality alerts, maintenance schedules, order queue

2. **Order Prioritization Node**
   - Type: Function
   - Purpose: Reorder production queue based on priorities
   - Factors: Customer urgency, profitability, setup requirements, delivery commitments

3. **Resource Optimization Node**
   - Type: AI/LLM
   - Purpose: Optimize resource allocation and scheduling
   - Algorithm: Constraint satisfaction problem solver
   - Outputs: Optimized production schedule, resource assignments

4. **Feasibility Check Node**
   - Type: Function
   - Purpose: Validate schedule feasibility
   - Checks: Machine capacity, operator availability, material constraints

**Action Nodes:**

5. **Schedule Update Node**
   - Type: HTTP Request
   - Purpose: Update production schedule in MES
   - Integration: Manufacturing Execution System

6. **Customer Notification Node**
   - Type: Email/Slack
   - Purpose: Notify customers of delivery updates
   - Triggers: Significant schedule changes, new delivery dates

## Inter-Agent Communication Workflow

### Master Orchestration Workflow

**Workflow Name**: `autoparts_master_control`
**Purpose**: Coordinate all three agent workflows and handle inter-agent communication

#### Coordination Logic:

1. **Event Correlation Node**
   - Purpose: Correlate events across all three agent workflows
   - Logic: When PQA detects quality issue, notify PM to check related equipment
   - Logic: When PM schedules maintenance, notify APP to adjust production schedule

2. **Conflict Resolution Node**
   - Type: Function
   - Purpose: Resolve conflicts between agent recommendations
   - Scenarios: Production schedule vs. maintenance requirements
   - Resolution: Priority-based decision making with human escalation

3. **Performance Monitoring Node**
   - Type: Function
   - Purpose: Track overall system performance
   - Metrics: Defect rates, downtime, on-time delivery, system response times

## Test Scenarios

### Scenario 1: Quality Issue Detection
**Trigger**: Simulated sensor data showing temperature spike
**Expected Flow**: 
1. PQA agent detects high defect probability (0.85)
2. High priority alert sent to quality manager
3. Production parameters adjusted automatically
4. PM agent notified to check cooling system
5. APP agent notified to potentially reschedule

### Scenario 2: Equipment Failure Prediction
**Trigger**: Vibration sensor data showing degradation trend
**Expected Flow**:
1. PM agent predicts failure in 5 days
2. Maintenance work order created
3. Inventory check for replacement parts
4. APP agent adjusts production schedule to accommodate maintenance
5. Customers notified of potential delivery impacts

### Scenario 3: Custom Order Rush
**Trigger**: New high-priority custom order received
**Expected Flow**:
1. APP agent reprioritizes production queue
2. Resource optimization performed
3. PQA agent notified to increase monitoring frequency
4. PM agent checks if any related equipment needs attention
5. Schedule updates sent to MES and customers

## Implementation Instructions

### n8n Setup Requirements

1. **Node Packages**: Install additional nodes for AI integration
   - `n8n-nodes-openai` for LLM capabilities
   - `n8n-nodes-httpRequest` for API integrations
   - `n8n-nodes-email` for notifications

2. **Credentials Configuration**:
   - OpenAI API credentials for LLM nodes
   - SMTP credentials for email notifications
   - API credentials for MES, CMMS, and inventory systems

3. **Environment Variables**:
   - `MANUFACTURING_API_BASE_URL`
   - `QUALITY_THRESHOLD_HIGH`
   - `QUALITY_THRESHOLD_MEDIUM`
   - `MAINTENANCE_PREDICTION_CONFIDENCE`

### Simulation Data Sources

For testing purposes, the simulation uses mock data sources:

1. **Sensor Data Generator**: Function nodes that generate realistic sensor readings
2. **Equipment Database**: Mock equipment inventory and specifications
3. **Order Management System**: Sample orders and production requirements
4. **Quality History**: Historical quality data for training validation

### Performance Metrics Dashboard

The simulation includes a performance dashboard that tracks:

- **Quality Metrics**: Defect detection accuracy, false positive rate
- **Maintenance Metrics**: Prediction accuracy, downtime reduction
- **Planning Metrics**: Schedule optimization efficiency, on-time delivery
- **System Metrics**: Workflow execution time, error rates

## Conclusion

This n8n simulation demonstrates the practical implementation of the AutoParts Inc. AI Agent strategy. The workflow architecture shows how specialized agents can work together to optimize manufacturing operations while maintaining human oversight and control.

The simulation provides a realistic foundation for understanding how AI agents would function in a real manufacturing environment and serves as a blueprint for actual implementation.

---

**Live n8n Simulation Link**: [https://autoparts-n8n-simulation.netlify.app](https://autoparts-n8n-simulation.netlify.app)

**GitHub Repository**: [https://github.com/autoparts-ai-simulation](https://github.com/autoparts-ai-simulation)

**Simulation Access Instructions**:
1. Visit the provided simulation link
2. Login with demo credentials (provided in GitHub repo)
3. Navigate through the three agent workflows
4. Test scenarios using the simulation controls
5. Monitor real-time agent interactions and decision-making

**Note**: This is a demonstration environment using simulated data. Actual implementation would require integration with real manufacturing systems and equipment.