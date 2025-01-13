# AI Agent Workflow Specification

This document outlines the specification for defining AI agent workflows and 
agents using the Hive language. Hive uses a YAML-based syntax for clear and concise workflow definitions.

## Workflow Specification
Filename: <workflow_name>.hive.yml

Structure:
```yaml
version: 0.1.0
kind: workflow
name: <Workflow Name>
description: <Workflow Description>

trigger:
  event: <event_name>
  <event_specific_properties>: <value>

<stage_name>:
  agent: <agent_name>
  inputs:
    <input_name>: <value_or_reference>

<another_stage_name>:
  agent: <another_agent_name>
  inputs:
    <input_name>: <value_or_reference>
```

Example:
```
version: 0.1.0
kind: workflow
name: "Analyze Customer Feedback"
description: "This workflow analyzes customer feedback and generates a report."

trigger:
  event: file_uploaded
  file_type: csv

preprocess-data:
  agent: data-cleaner
  inputs:
    file_path: "${{ trigger.file_path }}"

sentiment-analysis:
  agent: sentiment-analyzer
  inputs:
    data: "${{ preprocess-data.outputs.cleaned_data }}"

report-generation:
  agent: report-generator
  inputs:
    sentiment_data: "${{ sentiment-analysis.outputs.sentiment_scores }}"
    report_type: summary
```

Fields:

* version: Must be set to a valid hive-o/spec release version to indicate syntax spec version.
* kind: Must be set to workflow/agent to indicate a workflow/agent specification file.
* name: A descriptive name for the workflow.
* description: (Optional) A brief description of the workflow's purpose.
* trigger: Defines the event that triggers the workflow execution.
  * event: The name of the triggering event (e.g., file_uploaded, message_received, schedule).
  * <event_specific_properties>: Event-specific properties, such as file_type for the file_uploaded event.
* <stage_name>: Defines a stage in the workflow. Stage names should be unique within the workflow.
  * agent: The name of the agent to execute in this stage. This name should match an agent defined in an agents.hive.agent.yml file.
  * inputs: Key-value pairs where the key is the input name and the value is either a literal value, a reference to a trigger property (e.g., ${{ trigger.file_path }}), or a reference to an output from a previous stage (e.g., ${{ preprocess-data.outputs.cleaned_data }}).

## Agent Specification
Filename: agents.hive.agent.yml

Structure:
```yaml
version: 0.1.0
kind: agent
<agent_name>:
  description: <Agent Description>
  code:
    runtime: <runtime_environment>
    entrypoint: <entrypoint_script>
    # ... other code-related properties (e.g., dependencies, environment variables)
  inputs:
    <input_name>:
      type: <data_type>
      description: <Input Description>
  outputs:
    <output_name>:
      type: <data_type>
      description: <Output Description>
```

Example:
```yaml
version: 0.1.0
kind: agent
data-cleaner:
  description: "Cleans and preprocesses data"
  code:
    runtime: python-3.9
    entrypoint: clean.py
  inputs:
    file_path:
      type: string
      description: "Path to the input data file"
  outputs:
    cleaned_data:
      type: list
      description: "List of cleaned data items"

sentiment-analyzer:
  description: "Performs sentiment analysis on text data"
  code: 
    runtime: python-3.9
    entrypoint: analyze.py
  inputs:
    data:
      type: list
      description: "List of text data to analyze"
  outputs:
    sentiment_scores:
      type: list
      description: "List of sentiment scores"
```

Fields:

* version: Must be set to a valid hive-o/spec release version to indicate syntax spec version.
* kind: Must be set to workflow/agent to indicate a workflow/agent specification file.
* <agent_name>: A unique name for the agent.
* description: (Optional) A brief description of the agent's purpose.
* code: Defines the custom code for the agent.
  * runtime: The runtime environment required to execute the code (e.g., python-3.9, nodejs-16).
  * entrypoint: The entrypoint script or file to be executed.
  * Other code-related properties: (Optional) Can include dependencies, environment variables, and other relevant information.
* inputs: Defines the expected inputs for the agent.
  * <input_name>: The name of the input.
    * type: The data type of the input (e.g., string, number, list).
    * description: (Optional) A description of the input.
* outputs: Defines the expected outputs produced by the agent.
  * <output_name>: The name of the output.
    * type: The data type of the output.
    * description: (Optional) A description of the output.

## Data Types
The following data types are currently supported:

string
number
boolean
list

## Referencing Values
* Trigger Data: Access trigger properties using ${{ trigger.<property_name> }} (e.g., ${{ trigger.file_path }}).
* Previous Stage Outputs: Access outputs from previous stages using ${{ <stage_name>.outputs.<output_name> }} (e.g., ${{ preprocess-data.outputs.cleaned_data }}).

This specification provides a foundation for defining and managing AI agent workflows using the Hive language. You can extend this specification by adding support for more complex data types, conditional logic, loops, and other features as needed.
