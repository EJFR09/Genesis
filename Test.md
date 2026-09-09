                    MITIC
                      │
              autoriza source ref
                      │
                      ▼
                    GitLab
                      │
                      ▼
                   Jenkins
                      │
          Validaciones automáticas
          ├─ source ref
          ├─ versionado
          ├─ ambiente
          ├─ configuración
          └─ credenciales
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       Android                   iOS
      Linux/Docker              macOS
          │                       │
       Fastlane                Fastlane
          │                       │
          ▼                       ▼
     AAB firmado              IPA firmado
          │                       │
          ▼                       ▼
 Google Play Testing         TestFlight
          │                       │
          └─────────┬─────────────┘
                    ▼
                 QA / MITIC
                    │
                  OK
                    │
                    ▼
             autorización humana
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   Google Play Prod       App Review
          │                   │
          ▼                   ▼
       Play Store          App Store