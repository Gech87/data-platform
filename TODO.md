# TODO: Fix Airbyte Kubernetes Deployment Issues

## Issues to Address
1. **Missing volume definitions**: Server and worker deployments have volumeMounts but no 'volumes' section.
2. **PVC naming inconsistencies**: Ensure PVC names match between volumes.yaml and deployments.
3. **Outdated Airbyte version**: Update from 0.50.4 to latest (0.57.0).
4. **Name prefix handling**: Ensure references work with dev- prefix in overlay.
5. **Resource conflicts**: Verify no conflicts between main postgres and Airbyte db.

## Steps to Complete
- [ ] Update Airbyte image versions to 0.57.0 in all deployment files.
- [ ] Add missing 'volumes' sections to server and worker deployments referencing correct PVCs.
- [ ] Standardize PVC names in volumes.yaml if needed.
- [ ] Verify service references in deployments (should work with Kustomize prefixing).
- [ ] Check for port/resource conflicts between main postgres and Airbyte db.
- [ ] Test deployment after changes.
