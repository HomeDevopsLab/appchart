# Changelog

## [3.1.0] - 2026-04-05

### Added

- Adding ServiceAccountName option, to give certain application special permissions.

## [3.0.0] - 2025-12-30

### Added

- Ability to mount secrets as volumes. I'ts handy when you need to mount encrypted config file
- Ability to mount configmap as volume just as the other type of volume.

### Changed

- [BREAKING] volumes structure. Flag `enabled` was removed, along with `type` structure. Now volumes structure is flattened.
- [BREAKING] kubernetes service name is no longer combined with application name (.Release.Name). HelmRelese configuration will be more straightforward.
- [BREAKING] mountPath and subPath options are now child-nodes for volumes.nfs, volumes.configmap and volumes.secret
- Ingress name is now the same as application name.

### Removed

- [BREAKING] database.enabled flag
- Obsolete dbsecrets pre-install hook which was designed to generate databae credentials for application
- Faulty initwebsitedir pre-install hook, which was designed to create application directory on persistent storage
- Obsolete mattermos-notify post-hook. This hook was designed to send chat notification after application deployment
- Obsolete MySQLDBHelper pre-install hook which was designed to create mysql database for an application

## [2.7.0] - 2025-12-14

### Added

- Ability to insert custom securityContext block
- ConfigMap support

## [2.6.1] - 2025-02-15

### Fixed

- Namespace value is dynamically parsed from HelmRelease

## [2.6.0] - 2024-10-19

### Changed

- Startup and Liveness probes are moved to HelmRelease file
- initWebsiteDir hook: create website dir when it does not exist

## [2.5.0] - 2024-09-01

### Added

- common rootdir on shared storage

## [2.4.0] - 2024-08-24

### Added

- livenessProbe (tcp)
- tolerations: wait 10s when node is unavaialble
- static loadBalancerIP for certain service

## [2.3.0] - 2024-08-05

### Changed

- removed liveness probes
- redefined startup probes

### Added

- private container registries support (imagePullSecrets)
- support of multiple ingresses to single deployment

## [2.2.0] - 2023-08-11

### Added

- option to run containers with additional command and args.

## [2.1.0] - 2023-07-27

### Added

- replaced strict TCP protocol family in ports specification with variable: protocol which allows to decide which protocol is needed

## [2.0.0] - 2023-07-21

### Added

- ability to use nodeSelector function

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
