graph TD
    A[Flow Engine Calls<br/>SendMobileAppMessageInvocableAction.invoke] --> B[Log Invocation Start<br/>@WithSpan creates root span]
    
    B --> C{Debug Mode Check}
    C -->|Yes| D[Return Mock Response]
    C -->|No| E[CMSMetadataService.getMobileAppBulkMessage]
    
    E --> F[AdminService.getCoreTenantId<br/>Convert off-core to core tenant ID]
    F --> G[Cache Check: mobileAppBulkMessageCache]
    G -->|Cache Hit| H[Return Cached BulkMessage]
    G -->|Cache Miss| I[Core API Call: BulkMessage Query]
    
    I --> J[Core Query API<br/>/services/data/v{apiVersion}/query<br/>SELECT Id,FlowRecordElementId,Name,Sender,CampaignId,Campaign.Name<br/>FROM BulkMessage WHERE FlowRecordElementId='{flowElementId}']
    
    J --> K[Parse Response & Create BulkMessage Object]
    K --> L[Create SendContext<br/>E360SendMobileAppService.SendContext]
    
    L --> M[Call e360SendMobileAppService.send]
    
    M --> N[Initialize SendMobileAppLog<br/>Start timing]
    
    N --> O[Process Requests for Recipients]
    
    O --> P[For Each Request:<br/>1. fetchUnifiedIndividualIdFromRequest<br/>2. Try Data Graph First]
    
    P --> Q{Data Graph Available?}
    Q -->|Yes| R[E360MobileAppDataGraphUtil.createRecipientsFromDataGraph<br/>Process in-memory data]
    Q -->|No| S[Mark for External Fetch]
    
    R --> T[Add to ProcessingResults<br/>Increment numRequestsUsingDataGraph]
    S --> U[Add to requestsNeedingFetch Map]
    
    U --> V[Batch Process External Sources]
    V --> W[GaterService.getBooleanValue<br/>Feature Flag: useHsoForContactPointFetch]
    
    W --> X{Use HSDB?}
    X -->|Yes| Y[fetchFromHSDB]
    X -->|No| Z[fetchFromCDP]
    
    Y --> AA[E360MobileAppCDPQueryUtil.fetchIndividualIdsFromCDP<br/>CDP API: Unified Individual → Individual ID mapping]
    AA --> BB[CoreHighScaleObjectsService.getMobileDeviceAppRegistrations<br/>Core API: Mobile Device App Registration]
    
    BB --> CC[Core Query API<br/>/services/data/v{apiVersion}/query<br/>SELECT DeviceId,DeviceSystemToken,DevicePlatform,MobileAppId<br/>FROM MobileDeviceAppRegistration<br/>WHERE DeviceId IN ({deviceIds}) AND MobileAppId='{mobileAppId}']
    
    Z --> DD[E360MobileAppCDPQueryUtil.parseRecipientInfoFromCDP<br/>CDP API: Direct recipient info fetch]
    
    CC --> EE[Convert Registrations to PushRecipients<br/>Set device tokens, platform info]
    DD --> FF[Process CDP Response<br/>Increment numRequestsUsingCDP]
    
    EE --> GG[Add to ProcessingResults<br/>Increment numRequestsUsingHSDB]
    FF --> GG
    
    GG --> HH[Build RecipientResult List<br/>Map request index to recipients]
    
    HH --> II{Any Successful Recipients?}
    II -->|No| JJ[Skip API Call<br/>Set apiCallMade=false
