flowchart TD
    A["1. Entry Point:<br/>SendMobileAppMessageInvocableAction.invoke()"] --> B["2. Bulk Message Metadata Retrieval"]
    
    B --> B1["2a. AdminService.getCoreTenantId()<br/>Protocol: gRPC<br/>Purpose: Convert tenant ID<br/>Input: a360/testcdp002/...<br/>Output: core/falcontest1-core3sdb11/..."]
    B1 --> B2["2b. Core API - BulkMessage Query<br/>Protocol: REST<br/>Endpoint: /services/data/v/query<br/>Duration: 64ms<br/>Status: 200 OK"]
    
    B2 --> C["3. E360SendMobileAppServiceImpl.send()<br/>Main Processing"]
    
    C --> D["4. Recipient Processing:<br/>processRequestsForRecipients()"]
    
    D --> E["5. Data Graph Processing<br/>Tries to get recipients from data graph first<br/>No API calls - In-memory processing"]
    
    E --> F{Data Graph Success?}
    F -->|Yes| K["10. Payload Generation"]
    F -->|No| G["6. External Source Fetching"]
    
    G --> H["7. Gater Service - Feature Flag Check<br/>Protocol: gRPC<br/>Port: 10443<br/>Gate: useHsoForContactPointFetch<br/>Duration: 0.301ms<br/>Timeout: 4000ms"]
    
    H --> I{HSDB Enabled?}
    
    I -->|No| J1["8. CDP Fetching (HSDB disabled)<br/>Protocol: gRPC<br/>Service: ansiSqlQueryStream<br/>Timeout: 3600s<br/>Single comprehensive query<br/>Duration: 10-50ms"]
    
    I -->|Yes| J2["9. HSDB Fetching (HSDB enabled)"]
    
    J2 --> J2A["9a. CDP Query - Individual ID Mapping<br/>Protocol: gRPC<br/>Service: ansiSqlQueryStream<br/>Query: SELECT SourceRecordId, UnifiedRecordId<br/>FROM UnifiedLinkIndividualDlm<br/>Duration: ~10ms"]
    
    J2A --> J2B["9b. Mobile Device App Registration Query<br/>Protocol: REST<br/>Endpoint: /services/data/v/query<br/>Query: SELECT DeviceId, DeviceSystemToken<br/>FROM MobileDeviceAppRegistration<br/>Duration: 77ms"]
    
    J1 --> K
    J2B --> K
    
    K["10. Payload Generation:<br/>sendPushNotifications()"]
    
    K --> L["11. OMM API Call<br/>Protocol: REST<br/>Endpoint: /api/v2/push/messages<br/>Method: POST<br/>Duration: 433ms<br/>Auth: Near-core auth provider<br/>Payload: JSON with recipients array"]
    
    L --> M["End: Response Processing"]
    
    style A fill:#e3f2fd
    style B fill:#f3e5f5
    style B1 fill:#fff3e0
    style B2 fill:#fff3e0
    style E fill:#e8f5e8
    style H fill:#fff3e0
    style J1 fill:#fff3e0
    style J2A fill:#fff3e0
    style J2B fill:#fff3e0
    style L fill:#ffebee
    style M fill:#e8f5e8
