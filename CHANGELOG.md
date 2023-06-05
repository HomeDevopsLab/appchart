# Changelog

## [1.0.3] - 2023-06-05
### Chaged
- increased failure threshold
## [1.0.2] - 2023-05-25

### Fixed

- hook deployment errors

## [1.0.0] - 2023-05-25

### Added

- startup probes
- Mattermost Notifications

## [0.1.0] - 2023-05-06

### Changed

- Treat other than http service as ClusterIP type
- NodePort type service is now obsolete

## [0.0.1] - 2023-04-15

### Added

- Create website dirs pre-install hook
- Create secret and mysql database pre-install hook
- Support of different types of volumes: longhorn pvc and nfs
- Support of ingress with SSL
- Standarized HelmRelease file
