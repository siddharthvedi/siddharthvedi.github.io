# In-App Message (IAM) Download Analytics Flow - End-to-End Diagram

```mermaid
graph TD
    %% Mobile App Side
    A[Mobile App] --> B[In-App Message Download]
    B --> C[SDK Detects Download]
    C --> D[InternalAnalyticManager.onInAppMessageDownloaded]
    
    %% Android SDK Analytics Processing
    D --> E[BillableEtAnalytics]
    D --> F[EtAnalytic]
    D --> G[PiWamaAnalytic]
    D --> H[DeviceStats]
    
    E --> I[Create AnalyticItem]
    F --> I
    G --> I
    H --> I
    
    I --> J[Store in Local Analytics Storage]
    J --> K[EtAnalyticsSender Batch Processing]
    K --> L[HTTP Request to Server]
    
    %% Server-Side Reception
    L --> M[StatsConsumer]
    M --> N{Message Type?}
    N -->|pushinappdownload| O[ProcessMobilePushInAppDownload]
    N -->|Other| P[Other Processing]
    
    %% Event Creation & Queue
    O --> Q[MobilePushInAppDownloadEvent]
    Q --> R[Create NameValueCollection]
    R --> S[QueueMessage to Channel: mobilepush]
    
    %% Database Storage
    S --> T[StatsStaging Database]
    T --> U[PushInAppDownloadEvent Table]
    
    %% Batch Processing
    U --> V[BatchStatWriter]
    V --> W[PushInAppDownloadInsBatch Stored Procedure]
    W --> X[Insert into PushInAppDownloadEvent]
    
    %% Post-Processing Pipeline
    X --> Y[PushStatsStagingSlotWorker]
    Y --> Z[PushInAppActivityStatisticsProcessor]
    Y --> AA[PushInAppBillingProcessor]
    Y --> BB[PushInAppPresentedEventProcessor]
    Y --> CC[PushInAppDownloadValidationEventProcessor]
    
    %% Validation Error Handling
    CC --> DD[GetNextBatchOfValidationErrorEvents]
    DD --> EE[SetMessageSuppression]
    DD --> FF[TrackEvents]
    
    %% Data Retention
    X --> GG[90-Day Retention Policy]
    GG --> HH[DeprecatePushInAppDownloadEvent]
    
    %% Reporting & Analytics
    X --> II[Reporting Database]
    II --> JJ[Analytics Dashboards]
    II --> KK[Billing Reports]
    II --> LL[Operational Reports]
    
    %% Error Handling Path
    C --> MM[Download Validation Error]
    MM --> NN[MobilePushInAppDownloadValidationErrorEvent]
    NN --> OO[PushInAppDownloadValidationErrorEvent Table]
    OO --> PP[Validation Error Processing]
    
    %% Styling
    classDef mobileApp fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef sdk fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef server fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef database fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef processing fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef error fill:#ffebee,stroke:#c62828,stroke-width:2px
    classDef reporting fill:#e0f2f1,stroke:#004d40,stroke-width:2px
    
    class A,B mobileApp
    class C,D,E,F,G,H,I,J,K,L sdk
    class M,N,O,Q,R,S,T,V,W,X,Y,Z,AA,BB,CC,DD,EE,FF server
    class U,OO database
    class GG,HH,II,JJ,KK,LL processing
    class MM,NN,PP error
    class JJ,KK,LL reporting
```

## Flow Description

### **Phase 1: Mobile App & SDK (Blue/Purple)**
1. **Mobile App**: User's mobile application
2. **In-App Message Download**: IAM content is downloaded to device
3. **SDK Detects Download**: Android SDK detects the download event
4. **Analytics Manager**: `InternalAnalyticManager` processes the event
5. **Multiple Listeners**: Billable, ET, PI, and Device stats listeners
6. **Analytics Item**: Creates `AnalyticItem` with event data
7. **Local Storage**: Stores analytics data locally
8. **Batch Processing**: `EtAnalyticsSender` batches and sends data
9. **HTTP Request**: Sends analytics to server

### **Phase 2: Server Reception (Green)**
10. **StatsConsumer**: Receives analytics messages
11. **Message Processing**: Routes "pushinappdownload" messages
12. **Event Creation**: `MobilePushInAppDownloadEvent` creates event
13. **Data Collection**: Creates `NameValueCollection` with metadata
14. **Queue Message**: Queues for processing with 'mobilepush' channel

### **Phase 3: Database Storage (Orange)**
15. **StatsStaging Database**: Primary analytics database
16. **PushInAppDownloadEvent Table**: Stores download events
17. **Batch Processing**: `BatchStatWriter` handles batching
18. **Stored Procedure**: `PushInAppDownloadInsBatch` inserts data
19. **Data Insertion**: Inserts into `PushInAppDownloadEvent` table

### **Phase 4: Post-Processing (Pink)**
20. **Slot Worker**: `PushStatsStagingSlotWorker` processes events
21. **Activity Statistics**: Processes activity data
22. **Billing**: Handles billing calculations
23. **Presented Events**: Processes display events
24. **Validation Errors**: Handles download validation errors

### **Phase 5: Error Handling (Red)**
25. **Validation Errors**: Download validation failures
26. **Error Events**: `MobilePushInAppDownloadValidationErrorEvent`
27. **Error Table**: `PushInAppDownloadValidationErrorEvent` table
28. **Error Processing**: Specialized error handling

### **Phase 6: Data Management (Light Pink)**
29. **Retention Policy**: 90-day data retention
30. **Deprecation**: Automatic data cleanup
31. **Reporting**: Data flows to reporting systems
32. **Analytics**: Dashboard and reporting data
33. **Billing**: Billing report generation
34. **Operations**: Operational reporting

## Key Data Points Tracked

| Field | Description | Source |
|-------|-------------|---------|
| `DeviceID` | Unique device identifier | Mobile SDK |
| `ApplicationID` | App identifier | Mobile SDK |
| `PushMessageID` | In-app message ID | Server |
| `ActivityInstanceID` | Journey activity instance | Server |
| `EventDateUTC` | Download timestamp | Mobile SDK |
| `RequestID` | Request tracking ID | Mobile SDK |
| `UUID` | Unique event identifier | Mobile SDK |
| `MID/EID` | Member/Enterprise ID | Server |
| `CreatedBy` | User who triggered event | Server |

## Error Handling

- **Download Validation Errors**: Tracked separately in `PushInAppDownloadValidationErrorEvent`
- **Network Failures**: Retry mechanisms in `EtAnalyticsSender`
- **Database Failures**: Batch processing with rollback capabilities
- **Data Retention**: Automatic cleanup after 90 days

## Performance Considerations

- **Batch Processing**: Analytics are batched for efficiency
- **Asynchronous Processing**: Non-blocking event processing
- **Database Optimization**: Indexed tables for fast queries
- **Retention Policies**: Automatic data cleanup to manage storage
