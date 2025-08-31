sequenceDiagram
    participant FE as Flow Engine
    participant SMAMIA as SendMobileAppMessageInvocableAction
    participant CMS as CMSMetadataService
    participant AS as AdminService
    participant CoreAPI as Salesforce Core API
    participant GS as Gater Service
    participant E360 as E360SendMobileAppService
    participant CDP as Customer Data Platform
    participant HSDB as High Scale Database
    participant OMM as OMM LLTS API

    Note over FE, OMM: Mobile App Message Send Flow

    FE->>SMAMIA: invoke()
    
    SMAMIA->>CMS: getMobileAppBulkMessage()
    
    CMS->>AS: getCoreTenantId()
    Note right of AS: Convert off-core to core tenant ID<br/>Input: a360/testcdp002/5d6cad694fe848eb99776014f97e30a2<br/>Output: core/falcontest1-core3sdb11/00DSN000000TBad2AG
    AS-->>CMS: CoreTenantId
    
    CMS->>CoreAPI: GET /services/data/v{apiVersion}/query
    Note right of CoreAPI: BulkMessage Query<br/>Duration: 64ms<br/>Status: 200 OK
    CoreAPI-->>CMS: BulkMessage metadata
    
    CMS-->>SMAMIA: BulkMessage object
    
    SMAMIA->>E360: send()
    
    E360->>GS: getBooleanValue()
    Note right of GS: Feature Flag Check<br/>Gate: com.salesforce.e360.push.useHsoForContactPointFetch<br/>Value: true<br/>Duration: 0.301ms
    GS-->>E360: true
    
    E360->>CDP: queryDataCloudStreaming()
    Note right of CDP: Individual ID Mapping<br/>SELECT "SourceRecordId__c" AS IndividualId,<br/>"UnifiedRecordId__c" AS UnifiedIndividualId<br/>FROM UnifiedLinkssotIndividualIr01__dlm<br/>WHERE "UnifiedRecordId__c" IN<br/>('064d39b466d3d4b668aaab683de33638')
    CDP-->>E360: Individual IDs
    
    E360->>HSDB: getMobileDeviceAppRegistrations()
    HSDB->>CoreAPI: GET /services/data/v{apiVersion}/query
    Note right of CoreAPI: Mobile Device App Registration Query<br/>Duration: 77ms<br/>SELECT DeviceId,DeviceSystemToken,DevicePlatform,MobileAppId<br/>FROM MobileDeviceAppRegistration<br/>WHERE DeviceId IN ({deviceIds})<br/>AND MobileAppId='{mobileAppId}'
    CoreAPI-->>HSDB: Device registrations
    HSDB-->>E360: PushRecipients
    
    E360->>OMM: POST /api/v2/push/messages
    Note right of OMM: Push Notification Send<br/>Duration: 433ms<br/>definitionKey: 539SN000000PcOe<br/>messageKey: 023f036512954fb9dbab08a8a9a6fffda5b89a3e<br/>platform: iPhone OS<br/>individualId: C5450D62-5B63-4C56-9A49-768E227971AE<br/>bulkMessageId: 539SN000000PcOeYAK
    OMM-->>E360: SendPushNotificationResponse
    
    E360-->>SMAMIA: Response list
    
    SMAMIA-->>FE: InvocationResult
