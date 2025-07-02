# In-App Message (IAM) Download Analytics - Server-Side Flow Diagram

```mermaid
graph TD
    %% SDK to Server
    A[SDK sends ET_ANALYTICS request] --> B[AnalyticEventServiceImpl.RecordAnalyticEvents]
    
    %% Server Processing
    B --> C[DeviceAnalyticEvent.ProcessAnalyticEvents]
    C --> D{Event Type Check}
    D -->|pushinappdownload| E[ProcessMobilePushInAppDownload]
    
    %% Event Processing
    E --> F[Create MobilePushInAppDownloadEvent]
    F --> G[StatsConsumer.ProcessEvent]
    G --> H[ProcessStats.ProcessMobilePushInAppDownload]
    
    %% Batching and Database
    H --> I[BatchPushInboxDownloadStatWriter]
    I --> J[Create PushInAppDownloadInteractionTable]
    J --> K[Execute PushInAppDownloadInsBatch SP]
    K --> L[Insert into PushInAppDownloadEvent table]
    
    %% Staging Table
    L --> M[StatsStaging Database]
    
    %% Styling
    classDef sdkClass fill:#e1f5fe
    classDef serverClass fill:#f3e5f5
    classDef processClass fill:#e8f5e8
    classDef databaseClass fill:#fff3e0
    
    class A sdkClass
    class B,C,D,E,F,G,H,I,J,K serverClass
    class L,M databaseClass
```

## Flow Description

### 1. **SDK Request Reception**
- SDK sends analytics data via `MCRequests.ET_ANALYTICS` endpoint
- `AnalyticEventServiceImpl.RecordAnalyticEvents()` receives the request

### 2. **Event Processing**
- `DeviceAnalyticEvent.ProcessAnalyticEvents()` processes the incoming events
- Event type is checked for `pushinappdownload`
- `ProcessMobilePushInAppDownload()` handles the specific event type

### 3. **Event Object Creation**
- `MobilePushInAppDownloadEvent` object is created with event data
- Event is passed to `StatsConsumer.ProcessEvent()`

### 4. **Stats Processing**
- `ProcessStats.ProcessMobilePushInAppDownload()` processes the event
- `BatchPushInboxDownloadStatWriter` handles batching for database insertion

### 5. **Database Insertion**
- `PushInAppDownloadInteractionTable` type is created with event data
- `PushInAppDownloadInsBatch` stored procedure is executed
- Data is inserted into `PushInAppDownloadEvent` table in StatsStaging database

### 6. **Final Storage**
- Event data is successfully stored in the staging table
- Ready for further processing and aggregation
