# Semantic Versioning

ReactiveUI uses [Semantic Versioning (SemVer)](http://semver.org/), adopting the use of major.minor.patch versioning, using the various parts of the version number to describe the degree and kind of change.

### Versioning Form

MAJOR.MINOR.PATCH[-ALPHA-BUILDNUMBER]

### Decision Tree

MAJOR when:
  - drop support for a platform
  - adopt a newer MAJOR version of an existing dependency 
  - disable a compatibility quirk off by default

MINOR when:
  - add public API surface area 
  - add new behavior
  - adopt a newer MINOR version of an existing dependency
  - introduce a new dependency 
  
PATCH when:
  - make bug fixes
  - add support for a newer platform
  - adopt a newer PATCH version of an existing dependency
  - any other change (not otherwise captured)

When determining what to increment when there are multiple changes, choose the highest kind of change.


