# Relativity Analytics Indexes API

Relativity Analytics are enabled by analytics indexes. Analytics indexes define spatial relationships between documents and concepts measured by their distance in a multidimensional space. For background information about Relativity Analytics indexes, see [Relativity Documentation site](https://help.relativity.com/9.4/Content/Relativity/Analytics/Analytics_indexes.htm).

You can automate the creation of Analytics indexes using the Services API or the REST API. Both APIs enable you to create and build new indexes and delete existing indexes. You can also use the Index Manager REST service for cross-platform and browser-based applications.

After you build the index, you can use it in analytics searches through the Relativity UI or the API. For more information, see [AnalyticsSearch](../DTO_reference/AnalyticsSearch/AnalyticsSearch.htm).

This page contains the following sections:

  - [Analytics index fundamentals](#Fundamentals)
  - [Create an index](#Instantiate)
  - [Submit an index job](#Create2)
  - [Build Analytics indexes](#Update)
  - [Automation index job example](#Automati)
  - [Index Manager REST service](#Index)

## Analytics index fundamentals

Before programmatically interacting with analytics indexes, familiarize yourself with the Relativity user interface and documentation. Note there is a strong correlation between the API operations and object properties and the user interface elements. You must also ensure that your Relativity credentials have the necessary permissions to the Analytics objects.

To provide your users with an analytics index:

  - Make sure the index options are set correctly in the Analytics profile. If necessary, create a new profile using the Relativity UI. Note the Artifact ID of the profile.
  - Create the index.
  - Submit the index job to populate and build the index.

Note that you can also submit a job to incrementally update an existing index. There are different steps required to build a new index versus incrementally updating an existing index, corresponding to the Analytics Index console buttons in the Relation UI and the values defined by the [AnalyticsIndexJobType enum](#Analytic).

### Build a new index

1.  **Populate index** – add all documents from the training set and searchable set to the ready-to-index list.
2.  **Build index** – build the concept space.
3.  **Enable queries** – load the index to server memory and prepare it for use.
4.  **Activate** – make the index available for users. Specifically, it adds the index to the search drop-down on the Documents tab and to the right-click menu in the viewer.

### Incrementally update an index

1.  **Populate index (incremental)** – check the data sources for both the training set and the searchable set. If there are any new documents in either search, it adds them to the index. At this stage, the Analytics index can be queried against.
2.  **Disable queries** – disable the index so that it cannot be queried against or used for any analytics operations. Queries must be disabled in order to perform the build.
3.  **Build index** – build the concept space. If any training documents have been added, this step will take longer as the engine rebuilds the concept space. If only searchable documents have been added, the engine starts this build at a later step (updating searchable items) as it just needs to map the new searchable documents into the pre-existing concept space.
4.  **Enable queries** – load the index to server memory and allows it to be used for querying and operations.
5.  **Activate** – make the index available for users.

## Create an index

The AnalyticsIndex represents the Analytics index. Properties include:

  - **Order** – the order in which the index appears in dropdowns.
  - **AnalyticsProfile** – the analytics profile used by the index. The value is an AnalyticsProfileRef object.
  - **AnalyticsServer** – the analytics server to be used to build the index. The value is a ResourceServerRef object.
  - **Email Notification Recipients** – the list of emails recipients to be notified during index population and build.
  - **TrainingSet** – the saved search to populate the training set. The value is a SavedSearchRef object.
  - **SearchableSet** – the saved search to populate the searchable set. The value is a SavedSearchRef object.
  - **OptimizeTrainingSet** – indicates whether the training set should be optimized (select only conceptually relevant documents from the training set saved search).
  - **AutomaticallyRemoveEnglishEmailSignaturesAndFooters** – indicates whether email signatures should be automatically removed during index build.

To create an index:

1.  Like with other Services API interfaces, start by accessing the IIndexManager interface through a client proxy to the Services API and instantiating an IndexManager object. How you build the client proxy depends on how you plan to use it. If you plan to use it in a custom page, event handler, or agent, you use API Helpers:

    ```csharp
    Relativity.Services.Analytics.IIndexManager indexManager = this.Helper.GetServicesManager().CreateProxy<Relativity.Services.Analytics.IIndexManager>(ExecutionIdentity.System);
    ```

    Otherwise, instantiate the IndexManager object using the ServiceFactory class:

    ```csharp
    String restServerAddress = "http://localhost/relativity.rest/api";
    String rsapiServerAddress = "http://localhost/relativity.services/api";

    Uri keplerUri = new Uri(restServerAddress);
    Uri servicesUri = new Uri(rsapiServerAddress);

    Relativity.Services.ServiceProxy.ServiceFactorySettings settings = new Relativity.Services.ServiceProxy.ServiceFactorySettings(
    servicesUri, keplerUri, new Relativity.Services.ServiceProxy.UsernamePasswordCredentials("jsmith@mycompany.com", "ExamplePassword1!"));

    var serviceFactory = new Relativity.Services.ServiceProxy.ServiceFactory(settings);
    Relativity.Services.Analytics.IIndexManager indexManager = _serviceFactory.CreateProxy<IIndexManager>();
    ```

    For more information about creating proxies, see [Connect to the Services API](../Basic_concepts/Connect_to_the_Services_API.htm).

2.  Instantiate a WorkspaceRef object to specify the workspace where you want to create an index:

    ```csharp
    Relativity.Services.Workspace.WorkspaceRef workspaceRef = new Relativity.Services.Workspace.WorkspaceRef() { ArtifactID = 12345 };
    ```

3.  Instantiate the new AnalyticsIndex object:

    ```csharp
    Relativity.Services.Analytics.AnalyticsIndex indexDTO = new Relativity.Services.Analytics.AnalyticsIndex()
    {
    // The order in which the index appears in dropdowns
    Order = 999,

    // Define an AnalyticsProfileRef corresponding to the Analytics Profile you want to create the index with
    AnalyticsProfile = new Relativity.Services.Analytics.AnalyticsProfileRef() { ArtifactID = 12346 },

    // Define a ResourceServerRef corresponding to the Analytics Server you want to create the index on
    AnalyticsServer = new Relativity.Services.ResourceServer.ResourceServerRef() { ArtifactID = 12347 },

    // Saved Search to be used as a training set for the index. Artifact -1 corresponds to <Default Training Set>
    TrainingSet = new Relativity.Services.Search.SavedSearchRef() { ArtifactID = -1 },

    // Saved Search to be used as a searchable set for the index. Artifact -1 corresponds to <Default Searchable Set>
    SearchableSet = new Relativity.Services.Search.SavedSearchRef() { ArtifactID = -1 },

    // Other settings
    OptimizeTrainingSet = true,
    AutomaticallyRemoveEnglishEmailSignaturesAndFooters = true,
    EmailNotificationRecipients = new List<string> { "jsmith@mycompany.com", "jdoe@mycompany.com" },
    };
    ```

4.  Finally, create the index by calling the CreateAsync() method of the IIndexManager interface and specify the workspace and the index object:

    ```csharp
    var newAnalyticsIndex = indexManager.CreateAsync(workspaceRef, indexDTO).GetAwaiter().GetResult();
    ```

    You can now submit the index to be populated and built.

## Submit an index job

To submit an index job after you create the index:

1.  Get an instance of the IIndexManager interface as described in the previous section.

2.  Instantiate a WorkspaceRef object to specify the workspace as described in the previous section.

3.  Define the parameters of the index job using the AnalyticsIndexJobParameters object:

    ```csharp
    var parameters = new Relativity.Services.Analytics.AnalyticsIndexJobParameters()
    {
    JobType: "FullBuild",
    FinalAutomationStage: "ActivateIndex",
    AutomaticallyRemoveDocumentsInError: true
    };
    ```

4.  Submit the job by calling the SubmitJobAsync() method of the IIndexManager interface and specify the workspace, the index object, and job parameters object:

    ```csharp
     indexManager.SubmitJobAsync(workspaceRef, newAnalyticsIndex, parameters);
    ```

Note that depending on the specified job options, you may need to submit the job multiple times.

### Job validation and error states

When submitting an Analytics index job, the job type must be applicable to the current state of the index. For example, the Stop Population job can be run if the index is currently populating. There are many jobs that can be run on an analytics index, and there are many states that an analytics index can be in.

The states of an Analytics index are:

  - New
  - Job in Queue
  - Population Failed
  - Attempting To Cancel Population
  - Population Canceled By User
  - Populating
  - Building
  - Removing Documents in Error
  - Waiting On Population
  - Index Build Failed
  - Disabling Queries
  - Enabling Queries
  - Build Recommended
  - Enable Queries Recommended
  - Activation Recommended
  - Queries Enabled
  - Queries Disabled Or Inactive
  - Build Is Not Complete
  - Error
  - In Automation Mode

**Note:** some of these states overlap with each other and others have various sub-states depending on additional index metadata. For example, many states include an error state like “STATE – 1 or more documents in error status,” where STATE is Populating, Building, etc. There are various error scenarios, for example, when items linked to the index (saved searches, analytics profile) can't be accessed. Regardless of any additional error state, the validator just treats it as "Error" and subsequently only lets you run the two error resolution-type jobs - ResolveErrors or RemoveDocumentsInError.

## Delete an index

To delete an index:

1.  Get an instance of the IIndexManager interface as described above.

2.  Define a WorkspaceRef to specify the workspace as described above.

3.  Delete the index by calling the DeleteAsync() method of the IIndexManager interface and specify the workspace and the index (as the SearchIndexRef object):

    ``` csharp
    indexManager.DeleteAsync(workspaceRef, newAnalyticsIndex);
    ```

## Automation index job example

The following is a complete automation index job example. It demonstrates how to create an index and run a full build with automation. The index is automatically activated and subsequently deleted.

``` csharp
public void UsingTheIndexManagerService()
{
// Get a reference to the Index Manager
Relativity.Services.Analytics.IIndexManager indexManager = this.Helper.GetServicesManager().CreateProxy<Relativity.Services.Analytics.IIndexManager>(ExecutionIdentity.System);

// Define a WorkspaceRef corresponding to the workspace you want to create and index in
Relativity.Services.Workspace.WorkspaceRef workspaceRef = new Relativity.Services.Workspace.WorkspaceRef() { ArtifactID = 12345 };

// Define the new AnalyticsIndex DTO
Relativity.Services.Analytics.AnalyticsIndex indexDTO = new Relativity.Services.Analytics.AnalyticsIndex()
{
// The order in which the index appears in dropdowns
Order = 999,

// Define an AnalyticsProfileRef corresponding to the Analytics Profile you want to create the index with
AnalyticsProfile = new Relativity.Services.Analytics.AnalyticsProfileRef() { ArtifactID = 12346 },

// Define a ResourceServerRef corresponding to the Analytics Server you want to create the index on
AnalyticsServer = new Relativity.Services.ResourceServer.ResourceServerRef() { ArtifactID = 12347 },

// Saved Search to be used as a training set for the index. Artifact -1 corresponds to <Default Training Set>
TrainingSet = new Relativity.Services.Search.SavedSearchRef() { ArtifactID = -1 },

// Saved Search to be used as a searchable set for the index. Artifact -1 corresponds to <Default Searchable Set>
SearchableSet = new Relativity.Services.Search.SavedSearchRef() { ArtifactID = -1 },

// Other settings
OptimizeTrainingSet = true,
AutomaticallyRemoveEnglishEmailSignaturesAndFooters = true,
EmailNotificationRecipients = new List<string> { "test1@relativity.com", "test2@relativity.com" },
};

// Create the new index
var newAnalyticsIndex = indexManager.CreateAsync(workspaceRef, indexDTO).GetAwaiter().GetResult();

// Define the job you want to run
var parameters = new Relativity.Services.Analytics.AnalyticsIndexJobParameters()
{
JobType: "FullBuild",
FinalAutomationStage: "ActivateIndex",
AutomaticallyRemoveDocumentsInError: true
};

// Build the index
indexManager.SubmitJobAsync(workspaceRef, newAnalyticsIndex, parameters);

// Delete the index
indexManager.DeleteAsync(workspaceRef, newAnalyticsIndex);
}
```

## Index Manager REST service

The Index Manager service allows you to interact with analytics indexes from browser-based and cross-platform applications. The service provides the same set of operations as the IIndexManager .NET interface - create an index, submit an index job, and delete an index.

### Create an analytics index with REST

To create an analytics index from a RESTful application, send a POST request to the following Index Manager service URL. The workspace can be identified by Artifact ID or GUID:

```http
/Relativity.REST/api/Relativity.Analytics/workspaces/{workspaceArtifactID|workspaceGUID}/indexes
```

The request payload must include valid JSON representation of the index object:

```json
{
"index": {
"Order": 999,
"AnalyticsProfile": {
"ArtifactID": 1035661
},
"AnalyticsServer": {
"ArtifactID": 1931274
},
"TrainingSet": {
"ArtifactID": -1
},
"SearchableSet": {
"ArtifactID": -1
},
"OptimizeTrainingSet": true,
"AutomaticallyRemoveEnglishEmailSignaturesAndFooters": true,
"EmailNotificationRecipients": [
"test1@relativity.com",
"test2@relativity.com"
],
"Name": "Primary Index"
}
}
```

The response returns HTTP status code 201 for success and the created index object:

```json
{
"Order": 999,
"AnalyticsProfile": {
"ArtifactID": 1035661,
"Name": "Default"
},
"AnalyticsServer": {
"ArtifactID": 1931274,
"Name": "Analytics319",
"ServerType": {
"ArtifactID": 1015232
}
},
"EmailNotificationRecipients": [
"test1@relativity.com",
"test2@relativity.com"
],
"TrainingSet": {
"ArtifactID": -1,
"Name": "<default training set>"
},
"SearchableSet": {
"ArtifactID": -1,
"Name": "<default searchable set>"
},
"OptimizeTrainingSet": true,
"AutomaticallyRemoveEnglishEmailSignaturesAndFooters": true,
"ArtifactID": 1281225,
"Name": "Primary Index"
}
```

### Submit an analytics index job with REST

To submit an analytics index job from a RESTful application, send a POST request to the following Index Manager service URL. The workspace can be identified by Artifact ID or GUID:

```http
/Relativity.REST/api/Relativity.Analytics/workspaces/{workspaceArtifactID|workspaceGUID}/indexes/{indexArtifactID}/jobs
```

The request payload must include valid JSON representations of the index job parameters objects:

```json
{
"parameters": {
"JobType": "FullBuild",
"FinalAutomationStage": "ActivateIndex",
"AutomaticallyRemoveDocumentsInError": true
}
}
```

The response does not contain any data. Success or failure are indicated by the HTTP status code. For more information, see [HTTP status codes](../../REST_API/HTTP_status_codes.htm).

### Delete an analytics index with REST

To delete an index, issue a DELETE request to the following Index Manager service URL. The workspace can be identified by Artifact ID or GUID:

```http
/Relativity.REST/api/Relativity.Analytics/workspaces/{workspaceArtifactID|workspaceGUID}/indexes/{indexArtifactID}
```

The response does not contain any data. Success or failure are indicated by the HTTP status code.
