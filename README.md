# Parallel Background RE-Framework Template
UiPath workflows which doesn´t need to interact with ui elements can parallelize the occurring workload.
For this purpose I came up with the PABARE-Framework (acronym).

This framework can spawn workers up to a configurable limit. It´s datasource for a transaction is an queue in Orchestrator or an `object` from a [ConcurrentQueue<T>](https://docs.microsoft.com/de-de/dotnet/api/system.collections.concurrent.concurrentqueue-1?view=netframework-4.8). The business process can be implemented in the `Process.xaml` file like it would be in the RE-Framework  because it utilize the structure of the [RE-Framework](https://github.com/UiPath/ReFrameWork).

Besides the parallism it also tries to improve things that IMHO the RE-Framework lacks behind.

# Use Cases

I need to find some ;D

## How to install
PABARE-Framework is a template project. It doesn´t have a *project.json* file.
Instead this project contains a *template.json* file inside the *.local* folder.

To use this template, you can download the latest release from this repository and unzip the release into *%localappdata%/UiPath/ProjectTemplates* folder.
Restart Studio and you can create a new process from this template from now on.

## Processing data in parallel in one running robot
The main task of PABARE-Framework is to parallelize the workload of an queue.

For this purpose it introduced a config parameter `BusinessProcess_ParallelExecutions` (Excel file in sheet *Constants*). This parameter, as the name already implies, is used to control the number of workers.

It default value is set to 10. You may tweak this value to get better results.

PABARE-Framework can consume an queue from Orchestrator or can take concurrent from a ConcurrentQueue.
You can chose between this two mechanism by toggle the option `Framework_UseOrchestratorQueue`

### Transaction data from an Orchestrator queue
If `Framework_UseOrchestratorQueue`is set to true PABARE-Framework looks into an Orchestrator queue.
It invokes the file named *Framework/Transaction/Queue/GetTransactionDataFromQueue.xaml*.  

> You have to set `Framework_OrchestratorQueueName` to a valid queue name.

### Transaction data from a conurrent queue.
If `Framework_UseOrchestratorQueue`is set to false PABARE-Framework doesn´t look into the orchestrator queue.

It instead invokes *Framework/Transaction/DataStructure/InitTransactionData.xaml*. In this file you can initialize the ConcurrentQueue from a datasource your choise. One entry in the ConcurrentQueue is one TranactionItem. Please keep in mind changing the ConcurrentQueue to anything else that isn´t a thread-safe collection can cause unexcepted behavior.

If you filled the ConcurrentQueue the workers invoke the file *Framework/Transaction/DataStructure/InitTransactionData.xaml*.
In this file you have to create an QueueItem with the needed ItemInformation. Use "Add Transaction Item" activity to do so.

> Remember, you have to set `Framework_OrchestratorQueueName` to a valid queue name.
>> If you want to work completly without Orchestrator queues I recommend you to create an QueueItem by hand (`new UiPath.Core.QueueItem()`). This is much less work than changing "QueueItem" to DataRow or something like that and helps to integrate the process into a queue in an easy manner if Orchestrator comes avialable.


## Additional features
### Process references
One big downside of the original RE-Framework is, when you want to work with persistent connections or even with window variables you have to add the variable through the whole workflow or add it to the *Config* dictionary.

To reduce the refactoring workload PABARE-Framework uses a second dictionary named `ProcessReferences(Of String, Object)`.
In this you can store e.g. a persistent database connection, a connection from the Exchange-Scope activity or anything else.

This dictionary can be accessed from all worker (not thread safe. I do not recommend to write to it from inside of a worker).
With the inroduction of this dictionary the main workflow doesn´t need to be edited at all.

To cast the object inside the dictionary back to its normal type you can use the [Ctype](https://docs.microsoft.com/de-de/dotnet/visual-basic/language-reference/functions/ctype-function) function. (e.g. `Ctype(in_ProcessReferences('ExchangeConnection', ExchangeWebservice)`)

This dictionary can be accessed from any XAML-File.

### Worker references
Similar to process references PABARE-Framework provides a third dictionary named `WorkerReferences(Of String, Object)`.

This dictionary can be used to store information inside a worker. The information never leaves the worker. Every worker has its own dictionary.

This dictionary can be accessed from any XAML-File.

### Credential management
To improve the way to deal with credentials PABARE-Framework introduce a new excel sheet named *Credentials*. In this sheet you can add your credential names. The credentials will be loaded at process start inside `InitAllSettings`.

The credentials are stored inside a dictionary. To make modeling with the framework userfriendly the framework uses [System.Net.NetworkCredential](https://docs.microsoft.com/en-us/dotnet/api/system.net.networkcredential?view=netframework-4.8) to store the information inside the dictionary (`dictionary(Of String, System.Net.NetworkCredential)`. With this .NET Class the user can retrieve a password and a password as SecureString as the same time.

### Log messages
Log messages generated from the PABARE-Framework are placed in the "Messages" sheet to make it more clear what these messages meant for.

The old way the RE-Framework generated this messages was to concat the dynamic part to the static part of the message.
To be more open to other languages PABARE-framework uses the function [String.Format](https://docs.microsoft.com/de-de/dotnet/api/system.string.format?view=netframework-4.8) to let the developer decide where information should be placed.

## Downsides
 1. As the name implies PABARE-Framework should only be used if a process or sub process doesn´t require any ui interaction.
Automating ui´s aren´t really supported but not impossible.
 2. Debugging a whole process can be quite funny. UiPath Studio isn´t build with parallism in mind. So if you stop at a breakpoint Studio shows you the last executed activity which isn´t the activity you may have been expected.
 3. If you choose a big number for `BusinessProcess_ParallelExecutions` you may become issues with the connectivity to orchestrator and the service you want to automate.

## Development phase
This is still an early stage. Expect bugs and typos in this project.

The documentation need to be extended as well.

## Planned
- Make it suiteable to headless automation which uipath is experimenting with.
- Migrating some features back to RE-Framework
- Make an Workflow Template for UiPath Go

# Credit goes to:
- UiPath for providing the RE-Framework ([MIT License](https://github.com/UiPath/ReFrameWork/blob/master/LICENSE))
- Max (FlMa - co-worker from cronos automation GmbH) to bring this idea into my mind
