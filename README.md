# Microsoft.Diagnostic.EventFlow

## Introduction
The EventFlow library suite allows applications to define what diagnostics data to collect, and where they should be outputted to. Diagnostics data can be anything from performance counters to application traces.
It runs in the same process as the application, so communication overhead is minimized. It also has an extensibility mechanism so additional inputs and outputs can be created and plugged into the framework. It comes with the following inputs and outputs:

**Inputs**
- Trace (aka System.Diagnostics.Trace)
- EventSource 
 
**Outputs**
- StdOutput (console output)
- Application Insights
- Azure EventHub
- Elastic Search

The EventFlow suite supports .NET applications and .NET Core applications. It allows diagnostic data to be collected and transferred for applications running in these Azure environments:

- Azure Web Apps
- Service Fabric
- Azure Cloud Service
- Azure Virtual Machines


## Getting Started
The libraries are distributed as nuget packages. To quickly get started, the libraries need to be added to the project and configured

**Nuget installation**
1. In Visual Studio, open the project which the libraries are to be used
2. Go to Tools->Options, find the Nuget Package Manager->Package Sources
3. Add a new nuget source with this location: **\\\\ddfiles\Team\Public\AT-Warsaw\Nuget\LKG**
4. Open nuget package manager, choose the package source you just added, install the Microsoft.Diagnostic.EventFlow nuget package. This nuget will bring down several other EventFlow nugets as well.

**Configuration**
1. After the nuget packages are install, there should be a eventFlowConfig.json file added to the project. Open this file in Visual Studio.
2. The installation generates a default configuration. A few sections are commented, which some of the optional configuration elements that can be added. It has an input type of "Trace", which assumes the application uses System.Diagnostics.Trace. Here is what it looks like
```json
{
    "inputs": [
        // {
        //   "type": "EventSource",
        //   "sources": [
        //     { "providerName": "Microsoft-Windows-ASPNET" }
        //   ]
        // },
        {
            "type": "Trace",
            "traceLevel": "Warning"
        }
    ],
    "filters": [
        {
            "type": "drop",
            "include": "Level == Verbose"
        }
    ],
    "outputs": [
        // Please update the instrumentationKey.
        {
            "type": "ApplicationInsights",
            "instrumentationKey": "00000000-0000-0000-0000-000000000000"
        }
    ],
    "schemaVersion": "2016-08-11",
    // "healthReporter": {
    //   "type": "CsvHealthReporter",
    //   "logFileFolder": ".",
    //   "logFilePrefix": "HealthReport",
    //   "minReportLevel": "Warning",
    //   "throttlingPeriodMsec": "1000"
    // },
    // "settings": {
    //    "pipelineBufferSize": "1000",
    //    "maxEventBatchSize": "100",
    //    "maxBatchDelayMsec": "500",
    //    "maxConcurrency": "8",
    //    "pipelineCompletionTimeoutMsec": "30000"
    // },
    "extensions": []
}
```
3. If you wish to send diagnostics data to Application Insights, fill in the value for the instrumentationKey. If not, simply remove the output block for Application Insights
4. To add a StdOutput output, install the Microsoft.Diagnostic.EventFlow.Outputs.StdOutput nuget. Then add the following in the outputs array in eventFlowConfig.json:
```json
    {
        "type": "StdOutput"
    }
```
5. Make sure there is at least one output defined. Run your application and see your traces in console output, or Application Insights.

## Configuration Details
The EventFlow pipeline is built around three core concepts: inputs, outputs, and filters. The number of inputs, outputs, and filters depend on the need of diagnostics. The configuration 
also has a healthReporter and settings section for configuring settings fundamental to the pipeline operation. At last, the extensions section allows declaration of custom developed
plugins. These extension declarations act like references. On pipeline initialization, EventFlow will search in the extensions first for input, output, or filter implementations.

### Inputs
These define what data will flow into the engine. At least one input is required. Each input type has its own set of parameters.

#### Trace
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.Inputs.Trace**

This input listens to traces written with System.Diagnostics.Trace API. Here is an example showing all possible settings:
```json
{
    "type": "Trace",
    "traceLevel":  "Warning"
}
```
| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| type | "Trace" | Yes | Specifies the input type. For this input, it must be "Trace". |
| traceLevel | Critical, Error, Warning, Information, Verbose, All | No | Specifies the collection trace level. Traces with equal or higher severity than specified are collected. For example, if Warning is specified, then Critial, Error, and Warning traces are collected. Default is All. |

#### EventSource
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.Inputs.EventSource**

This input listens to EventSource traces. EventSource classes can be created in the application by deriving from the [System.Diagnostics.Tracing.EventSource](https://msdn.microsoft.com/en-us/library/system.diagnostics.tracing.eventsource(v=vs.110).aspx) class. Here is an example showing all possible settings:
```json
{
    "type": "EventSource",
    "sources": [
        {
            "providerName": "MyEventSource",
            "level": "Informational",
            "keywords": "0x7F"
        }
    ]
}
```

*Top Object*

| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| type | "EventSource" | Yes | Specifies the input type. For this input, it must be "EventSource". |
| sources | JSON array | Yes | Specifies the EventSource objects to collect. |

*Sources Object*

| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| providerName | provider name | Yes | This field specify what kind of input this is. For this input, it must be "EventSource" |
| level | Critial, Error, Warning, Informational, Verbose, LogAlways | No | Specifies the collection trace level. Traces with equal or higher severity than specified are collected. For example, if Warning is specified, then Critial, Error, and Warning traces are collected. Default is All. |
| keywords | An integer | No | A bitmask that specifies what events to collect. Only events with keyword matching the bitmask are collected, except if it's 0, which means everything is collected. Default is 0. |

### Outputs
Outputs define where data will be published from the engine. It's an error if there are no outputs defined. Each output type has its own set of parameters.

#### StdOutput
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.Outputs.StdOutput**

This output writes data to the console window. Here is an example showing all possible settings:
```json
{
    "type": "StdOutput"
}
```
| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| type | "StdOutput" | Yes | Specifies the output type. For this output, it must be "StdOutput". |

#### Event Hub
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.Outputs.EventHub**

This output writes data to the [Azure Event Hub](https://azure.microsoft.com/en-us/documentation/articles/event-hubs-overview/). Here is an example showing all possible settings:
```json
{
    "type": "EventHub",
    "eventHubName": "myEventHub",
    "connectionString": "Endpoint=sb://<myEventHubNamespace>.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<MySharedAccessKey>"
}
```
| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| type | "EventHub" | Yes | Specifies the output type. For this output, it must be "EventHub". |
| eventHubName | event hub name | No | Specifies the name of the event hub. |
| connectionString | connection string | Yes | Specifies the connection string for the event hub. The corresponding shared access policy must have send permission. If the event hub name does not appear in the connection string, then it must be specified in the eventHubName field. |

#### Application Insights
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.Outputs.ApplicationInsights**

This output writes data to the [Application Insights](https://azure.microsoft.com/en-us/documentation/articles/app-insights-overview/). Here is an example showing all possible settings:
```json
{
    "type": "ApplicationInsights",
    "instrumentationKey": "00000000-0000-0000-0000-000000000000"
}
```
| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| type | "ApplicationInsights" | Yes | Specifies the output type. For this output, it must be "ApplicationInsights". |
| instrumentationKey | GUID | Yes | Specifies the instrumentation key for the targeted Application Insights resource. The key is in the form of a GUID. The key can be found on the Application Insights blade in Azure Portal. |

#### Elastic Search
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.Outputs.ElasticSearch**

This output writes data to the [Elastic Search](https://www.elastic.co/products/elasticsearch). Here is an example showing all possible settings:
```json
{
    "type": "ElasticSearch",
    "indexNamePrefix": "app1-",
    "serviceUri": "https://myElasticSearchCluster:9200",
    "basicAuthenticationUserName": "esUser1",
    "basicAuthenticationPassword": "<MyPassword>",
    "eventDocumentTypeName": "diagData"
}
```
| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| type | "ElasticSearch" | Yes | Specifies the output type. For this output, it must be "ElasticSearch". |
| indexNamePrefix | string | No | Specifies the prefix to be used when creating the Elastic Search index. This prefix, together with the date of when the data was generated, will be used to form the name of the Elastic Search index. If not specified, a prefix will not be used. |
| serviceUri | URL:port | Yes | Specifies where the Elastic Search cluster is. This is needed for EventFlow to locate the cluster and send the data. |
| basicAuthenticationUserName | string | No | Specifies the user name used to authenticate with Elastic Search. To protect the cluster, authentication is often setup on the cluster. |
| basicAuthenticationPassword | string | No | Specifies the password used to authenticate with Elastic Search. This field should be used only if basicAuthenticationUserName is specified. |
| eventDocumentTypeName | string | Yes | Specifies the document type to be applied when data is written. Elastic Search allows documents to be typed, so they can be distinguished from other types. This type name is user-defined. |

### Filters
As data comes through the EventFlow pipeline, the application can add extra processing or tagging to them. These optional operations are accomplished with filters. Filters can transform, drop, or tag data with extra metadata, with rules based on custom expressions.
With metadata tags, filters and outputs operating further down the pipeline can apply different processing for different data. For example, an output component can choose to send only data with a certain tag. Each filter type has its own set of parameters.

#### drop
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.Core**

This filter discards all data that satisfies the include expression. Here is an example showing all possible settings:
```json
{
    "type": "drop",
    "include": "Level == Verbose || Level == Informational"
}
```
| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| type | "drop" | Yes | Specifies the filter type. For this filter, it must be "drop". |
| include | logical expression | Yes | Specifies the logical expression that determines if the action should apply to the event data or not. For information about the logical expression, please see section [Logical Expressions](#logical-expressions). |

#### metadata
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.Core**

This filter adds additional metadata to all data that satisfies the include expression. Here is an example showing all possible settings:
```json
{
    "type": "metadata",
    "metadata": "metric",
    "include": "ProviderName == MyEventProvider && Id == 3",
    "customTag1": "tag1",
    "customTag2": "tag2"
}
```
| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| type | "metadata" | Yes | Specifies the filter type. For this filter, it must be "metadata". |
| metadata | string | Yes | Specifies the metadata type. This field is used only if type is "metadata", so it shouldn't appear in other filter types. The metadata type is user-defined and is persisted along with metadata tag added to the event data. |
| include | logical expression | Yes | Specifies the logical expression that determines if the action should apply to the event data or not. For information about the logical expression, please see section [Logical Expressions](#logical-expressions). |
| *[others]* | string | No | Specifies custom properties that should be added along with this metadata object. When the event data is processed by other filters or outputs, these properties can be accessed. The names of these properties are custom-defined and the possible set is open-ended. For a particular filter, zero or more custom properties can be defined. In the example above, customTag1 and customTag2 are such properties. |

### Health Reporter
Every software component can generate errors or warnings the developer should be aware of. The EventFlow library is no exception. An EventFlow health reporter reports errors and warnings generated by any components in the EventFlow pipeline.
In what format the report is presented depends on the implementation of the health reporter. The EventFlow library suite includes two health reporters: CsvHealthReporter and ServiceFabricHealthReporter. The health reporter can be configured in the same configuration file.
If the health reporter section is omitted, the pipeline will default to the CsvHealthReporter and writes to a file in a default location. Each health reporter has its own set of parameters.

#### CsvHealthReporter
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.Core**

This health reporter writes all errors, warnings, and informational traces generated from the pipeline into a CSV file. Here is an example showing all possible settings:
```json
"healthReporter": {
    "type": "CsvHealthReporter",
    "logFileFolder": ".",
    "logFilePrefix": "HealthReport",
    "minReportLevel": "Warning",
    "throttlingPeriodMsec": "1000"
}
```
| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| type | "CsvHealthReporter" | Yes | Specifies the health reporter type. For this reporter, it must be "CsvHealthReporter". |
| logFileFolder | file path | No | Specifies a path for the CSV log file to be written. It can be an absolute path, or a relative path. If it's a relative path, then it's computed relative to the directory where the EventFlow core library is in. However, if it's an ASP.NET application, it's relative to the app_data folder. |
| logFilePrefix | file path | No | Specifies a prefix used for the CSV log file. CsvHealthReporter creates the log file name by combining this prefix with the date of when the file is generated. If the prefix is omitted, then a default prefix of "HealthReport" is used. |
| minReportLevel | Error, Warning, Message | No | Specifies the collection report level. Report traces with equal or higher severity than specified are collected. For example, if Warning is specified, then Error, and Warning traces are collected. Default is Error. |
| throttlingPeriodMsec | number of milliseconds | No | Specifies the throttling time period. This setting protects the health reporter from being overwhelmed, which can happen if a message is repeatedly generated due to an error in the pipeline. Default is 0, for no throttling. |

#### ServiceFabricHealthReporter
*Nuget Package*: **Microsoft.Diagnostics.EventFlow.ServiceFabric**

This health reporter reports health back to the Service Fabric runtime. Configuration of this reporter is controlled via the Service Fabric service manifest file. To use it, only the health reporter type name is needed. Here is an example:
```json
"healthReporter": {
    "type": "ServiceFabricHealthReporter"
}
```

### Pipeline Settings
The EventFlow configuration has settings allowing the application to adjust certain behaviors of the pipeline. These range from how many events the pipeline buffer, to the timeout the pipeline should use when waiting for an operation. If this section is omitted, the pipeline will use default settings.
Here is an example of all the possible settings:
```json
"settings": {
    "pipelineBufferSize": "1000",
    "maxEventBatchSize": "100",
    "maxBatchDelayMsec": "500",
    "maxConcurrency": "8",
    "pipelineCompletionTimeoutMsec": "30000"
}
```
| Field | Values/Types | Required | Description |
| :---- | :-------------- | :------: | :---------- |
| pipelineBufferSize | number | No | Specifies how many events the pipeline can buffer if the events cannot flow through the pipeline fast enough. This buffer protects loss of data in cases where there is a sudden burst of data. |
| maxEventBatchSize | number | No | Specifies the maximum number of events to be batched before the batch gets pushed through the pipeline to filters and outputs. The batch is pushed down when it reaches the maxEventBatchSize, or its oldest event has been in the batch for more than maxBatchDelayMsec milliseconds. |
| maxBatchDelayMsec | number of milliseconds | No | Specifies the maximum time that events are held in a batch before the batch gets pushed through the pipeline to filters and outputs. The batch is pushed down when it reaches the maxEventBatchSize, or its oldest event has been in the batch for more than maxBatchDelayMsec milliseconds. |
| maxConcurrency | number | No | Specifies the maximum number of threads that events can be processed. Each event will be processed by a single thread, by multiple threads can process different events simultaneously. |
| pipelineCompletionTimeoutMsec | number of milliseconds | No | Specifies the timeout to wait for the pipeline to shutdown and clean up. The shutdown process starts when the DiagnosePipeline object is disposed, which usually happens on application exit. |


## Logical Expressions
TODO

## Contribution
* Prereq
    * Visual Studio 2015 Update 3 or above

* Build

    Run the following commands for to get a clean repo:

        tools\scorch.cmd
        tools\restore.cmd
    
    Run the following command to build a release build:

        tools\buildRelease.cmd

* Test

    Run the following command:

        tools\runtest.cmd

* Before sending out pull request:

    Run the following command and verify there is no error:

        tools\localCheckinGate.cmd

 