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

    Note over FE, OMM: Complete Mobile App Message Send Flow

    FE->>SMAMIA: invoke(context, config, request)
    Note right of SMAMIA: 09:16:44.819<br/>Trace: 07b8e2e11c62e99863945f75fbabe018

    SMAMIA->>CMS: getMobileAppBulkMessage(tenantId, flowRecordElementId)
    
    CMS->>AS: getCoreTenantId(offCoreTenantId)
    Note right of AS: Convert a360/testcdp002/... to core/falcontest1-core3sdb11/...
    AS-->>CMS: CoreTenantId
    
    CMS->>CoreAPI: GET /services/data/v{apiVersion}/query
    Note right of CoreAPI: SELECT Id,FlowRecordElementId,Name,Sender,CampaignId,Campaign.Name<br/>FROM BulkMessage WHERE FlowRecordElementId='{flowRecordElementId}'<br/>Duration: 64ms<br/>09:16:44.884
    CoreAPI-->>CMS: BulkMessage metadata (200 OK)
    
    CMS-->>SMAMIA: BulkMessage object
    SMAMIA->>SMAMIA: log("Successfully retrieved bulk message metadata")
    Note right of SMAMIA: 09:16:44.884

    SMAMIA->>E360: send(request, mobileAppId, tenantId, sendContext)
    
    E360->>GS: getBooleanValue("useHsoForContactPointFetch")
    Note right of GS: Gate check: com.salesforce.e360.push.useHsoForContactPointFetch<br/>Value: true<br/>Duration: 0.301ms<br/>09:16:44.885
    GS-->>E360: true (use HSDB path)
    
    E360->>CDP: fetchIndividualIdsFromCDP()
    Note right of CDP: SELECT "SourceRecordId__c" AS IndividualId,<br/>"UnifiedRecordId__c" AS UnifiedIndividualId<br/>FROM UnifiedLinkssotIndividualIr01__dlm<br/>WHERE "UnifiedRecordId__c" IN ('064d39b466d3d4b668aaab683de33638')<br/>09:16:44.885
    CDP-->>E360: Individual ID mapping
    
    E360->>HSDB: getMobileDeviceAppRegistrations()
    HSDB->>CoreAPI: GET /services/data/v{apiVersion}/query
    Note right of CoreAPI: SELECT DeviceId,DeviceSystemToken,DevicePlatform,MobileAppId<br/>FROM MobileDeviceAppRegistration<br/>WHERE DeviceId IN ({deviceIds}) AND MobileAppId='{mobileAppId}'<br/>Duration: 77ms<br/>09:16:45.034
    CoreAPI-->>HSDB: Device registrations (200 OK)
    HSDB-->>E360: List of PushRecipients
    
    E360->>OMM: POST /api/v2/push/messages
    Note right of OMM: SendPushNotificationRequest<br/>Recipients: 1<br/>Duration: 433ms<br/>09:16:45.468
    OMM-->>E360: SendPushNotificationResponse (Success)
    
    E360->>E360: log("send_mobile_app", metrics)
    Note right of E360: Log SendMobileAppLog<br/>09:16:45.468
    
    E360->>E360: log("send_mobile_app_sent_status")
    Note right of E360: Log SendMobileAppSentStatusLog<br/>09:16:45.467
    
    E360-->>SMAMIA: List of Responses
    SMAMIA->>SMAMIA: log("Successfully processed 1 mobile app message requests")
    Note right of SMAMIA: 09:16:45.468
    
    SMAMIA-->>FE: InvocationResult
