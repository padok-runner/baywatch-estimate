# Item Types & MCO Base Rates

## Item types with base rates (jour-homme / mois)

| Item                                                                                 | jour-homme / mois |
| ------------------------------------------------------------------------------------ | ----------------- |
| Public Cloud Managed Kubernetes Cluster                                              | 0.25              |
| Public Cloud Managed Virtual Machine                                                 | 0.1               |
| Public Cloud Managed Hypervisor                                                      | 0.25              |
| Private Cloud Managed Kubernetes Cluster                                             | 0.5               |
| Private Cloud Managed Virtual Machine                                                | 0.1               |
| Private Cloud Managed Hypervisor                                                     | 0.5               |
| Off-the-shelf application (apps developed by a third party, e.g. mysql, argo, redis) | 0.5               |
| Custom application (apps maintained by the client)                                   | 0.25              |

## Notes

- These are base rates before applying size/complexity coefficients.
- Each item must be evaluated per environment (prod, non-prod, shared services, landing zone).
- Rates are indicative and should be adjusted based on specific client context.
