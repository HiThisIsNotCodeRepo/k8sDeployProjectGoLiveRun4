# kubernetes Note
## What kubernetes can solve?
1. Handle server down.
2. Handle program hang.
3. Scale
## kubernete cluster set up
3 Master, 2 nodes
### IP address list


| Description | ip | 
| -------- | -------- | 
| master1     | 159.138.107.83     | 
| master2     | 119.8.180.23     | 
| master3     | 159.138.106.70    | 
| node1     | 159.138.100.84    | 
| node2     | 114.119.173.94    | 
### kubernet dashboard
URL=`https://<master or node IP address>:31586`

Token:
```shell=
eyJhbGciOiJSUzI1NiIsImtpZCI6IlVwVDh2VVFQdWlZTlhINmxfVldFNEJmNVI0d1BtVzN1WTQtd00yTjNMZG8ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWo2bXJyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2Nzc5NWQ4MS1kNTMzLTRhZDMtYjg2My1mMmNjMzQxZjFmMWMiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.SHlDcET85YBzr828JcGnlweVQHEWPpCK2EKhal1Dxo19u6PEbX9vfm5QEuk8DGa4LM-V-mbDV6LesNyp9fp2j1Qe24EzYq1jAK7ZszL5xw1hElTlN_SEjt2c1H2E0rExXdIcMXP1biiGgpalAjRk0M5q-KVn1m4tz2KTpjoy_cSwa1XmZKEPKR9GEHi9fO-VMEgdHegsMJAT8V3appfV6231jO-PGYJm0nXRZI-ZyXO9PB4W9Y9c0UziEO8ATNH6WPdv-kZRrirSLPODDuy3iJBZXrcBj1l7_Alh_gX0Be5PjBiurxUDvbpXKipN1IxnKDm_CN2oGbFXn6R-T2ACTg
```
## Pod 3 probe
1. Startup Probe,execute only initializing.
2. Liveness Probe,if fail the pod will restart.
3. Readiness Probe, if fails no traffic.
2 & 3 suitable for quick check so we can handle unavailable pod more timely.For long check use 1. [Reference](https://stackoverflow.com/questions/65858309/why-do-i-need-3-different-kind-of-probes-in-kubernetes-startupprobe-readinessp)

