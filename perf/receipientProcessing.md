flowchart TD
    A[Start: processRequestsForRecipients] --> B[Initialize results list and requestsNeedingFetch map]
    
    B --> C[For each request i=0 to requests.size]
    
    C --> D[Extract unifiedIndividualId from request]
    
    D --> E{unifiedIndividualId extraction successful?}
    E -->|No| F[Add ProcessingResult with error to results]
    E -->|Yes| G[Try Data Graph Processing]
    
    F --> C
    
    G --> H[Call processDataGraphForRecipients]
    
    H --> I{Data Graph processing successful?}
    I -->|Yes| J[Add successful ProcessingResult to results]
    I -->|No| K[Add unifiedIndividualId to requestsNeedingFetch map]
    
    J --> L[Increment sendLog.numRequestsUsingDataGraph]
    K --> M[Continue to next request]
    L --> M
    
    M --> N{More requests to process?}
    N -->|Yes| C
    N -->|No| O{requestsNeedingFetch map empty?}
    
    O -->|Yes| P[Return results list]
    O -->|No| Q[Start External Fetch Process]
    
    Q --> R[Record externalFetchStart time]
    
    R --> S[Call fetchRecipientsFromExternalSources]
    
    S --> T[Gater Service: Check GATE_USE_HSDB_FOR_CONTACT_POINT_FETCH]
    
    T --> U{shouldFetchFromHSDB?}
    U -->|Yes| V[fetchFromHSDB path]
    U -->|No| W[fetchFromCDP path]
    
    V --> V1[CDP Query: fetchIndividualIdsFromCDP]
    V1 --> V2{CDP query successful?}
    V2 -->|No| V3[Return error for all requests]
    V2 -->|Yes| V4[HSDB Query: getMobileDeviceAppRegistrations]
    V4 --> V5{HSDB query successful?}
    V5 -->|No| V6[Return error for all requests]
    V5 -->|Yes| V7[Map registrations back to UnifiedIndividuals]
    V7 --> V8[Create PushRecipient objects]
    V8 --> V9[Update sendLog.numRequestsUsingHSDB]
    
    W --> W1[CDP Query: parseRecipientInfoFromCDP]
    W1 --> W2[Process CDP results for each UnifiedIndividual]
    W2 --> W3[Update sendLog.numRequestsUsingCDP]
    
    V3 --> X
    V6 --> X
    V9 --> X
    W3 --> X
    
    X[Add fetch results to main results list]
    
    X --> Y[Calculate sendLog.externalFetchTime]
    
    Y --> P[Return complete results list]
    
    style A fill:#e1f5fe
    style P fill:#c8e6c9
    style I fill:#fff3e0
    style U fill:#fff3e0
    style E fill:#ffebee
    style V2 fill:#ffebee
    style V5 fill:#ffebee
