# Changelog

## [1.2.0](https://github.com/kleinnconrad/ml.prediction.depothedging/compare/v1.1.1...v1.2.0) (2026-10-09)


### Features

* pre merge pipeline ([eb7df4e](https://github.com/kleinnconrad/ml.prediction.depothedging/commit/eb7df4e248945db6cb5664f4085e2190ed6974c6))

## [1.1.1](https://github.com/kleinnconrad/ml.prediction.depothedging/compare/v1.1.0...v1.1.1) (2026-09-16)


### Bug Fixes

* locking transitive dependencies ([b843fd4](https://github.com/kleinnconrad/ml.prediction.depothedging/commit/b843fd4fc41ed92d9372e968838129bfcd34d047))

## [1.1.0](https://github.com/kleinnconrad/ml.prediction.depothedging/compare/v1.0.0...v1.1.0) (2026-09-16)


### Features

* switching to pyproject.toml for this project ([3ef2b37](https://github.com/kleinnconrad/ml.prediction.depothedging/commit/3ef2b37ec48719ac6a88204dece83613e7f66a9e))

## [1.0.0](https://github.com/kleinnconrad/ml.prediction.depothedging/compare/v0.2.1...v1.0.0) (2026-08-19)


### ⚠ BREAKING CHANGES

* changed the approach from strictly optimizing for 5% ROI to maximizing the ROI while ensuring a min_return of 2% while assuming 1/2 standard deviation as worse case scenario

### Features

* changed the approach from strictly optimizing for 5% ROI to maximizing the ROI while ensuring a min_return of 2% while assuming 1/2 standard deviation as worse case scenario ([7b78866](https://github.com/kleinnconrad/ml.prediction.depothedging/commit/7b78866afb5fa478049f7b4d0730afbcabdfd97e))

## [0.2.1](https://github.com/kleinnconrad/ml.prediction.depothedging/compare/v0.2.0...v0.2.1) (2026-08-17)


### Bug Fixes

* changed the override of expected_returns = 0 of the existing titles to capm usage. excluded cash from capm and assigned it a beta of -0.025 ([683406d](https://github.com/kleinnconrad/ml.prediction.depothedging/commit/683406d097d3f86a1a8feda0b780247359bb2504))

## [0.2.0](https://github.com/kleinnconrad/ml.prediction.depothedging/compare/v0.1.1...v0.2.0) (2026-08-14)


### Features

* using CAPM for more realistic expected returns of the existing portfolio ([3b4eeb7](https://github.com/kleinnconrad/ml.prediction.depothedging/commit/3b4eeb73faa249aa909dfd63dbd974efc2919c64))

## [0.1.1](https://github.com/kleinnconrad/ml.prediction.depothedging/compare/v0.1.0...v0.1.1) (2026-08-14)


### Bug Fixes

* configure cash for beta in settings.yml and setting it to -0.025 to reflect inflation ([d7ca848](https://github.com/kleinnconrad/ml.prediction.depothedging/commit/d7ca8480a114344bf5cc791bbf0f77ac0eee874f))

## 0.1.0 (2026-08-13)


### ⚠ BREAKING CHANGES

* update readme for first release
* update readme for first release

### Features

* update readme for first release ([49f49bd](https://github.com/kleinnconrad/ml.prediction.depothedging/commit/49f49bd61d1fceb455a54b1b82b10b52a243a67d))
* update readme for first release ([01e20fc](https://github.com/kleinnconrad/ml.prediction.depothedging/commit/01e20fcc7d8e0ec708f9cbd54004736ae30ca853))
