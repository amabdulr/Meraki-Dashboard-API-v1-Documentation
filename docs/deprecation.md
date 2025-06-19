# Deprecation

## Overview

Cisco Meraki provides newer, more performant APIs over time to help developer use cases currently addressed by existing operations and API versions. When introducing these alternatives, Meraki:

1. Marks the deprecated operation or version as deprecated in the OAS.
2. Documents the alternatives to the operation or version.

Cisco Meraki may occasionally phase out API versions or operations. After a designated deprecation period, the deprecated versions or operations are discontinued.

We encourage developers to leverage the improvements and migrate their applications to the latest offerings. Developers are expected to migrate their applications to non-deprecated offerings before the sunset date for those offerings.

## Definitions

### Version vs. revision

An API version, or _major_ version, groups a set of API resources and operations. Cisco Meraki uses simple integers for versions, such as "v1." Cisco Meraki releases these versions infrequently and currently only uses "v1."

An API revision, or _minor_ version, introduces non-breaking improvements to an API, such as adding attributes or capabilities to extend existing operations within a major version. However, revisions within a single version are usually additive and transparent to clients built on the previous version.

Cisco Meraki releases a new API revision every month, summarizing all changes released since the last revision. Cisco Meraki API revision names follow the format "1.50.0," where "1" is the version, "50" represents the minor version ID, and "0" signifies the patch version.

> NB: While the industry refers to major and minor versions in semantic versioning, this guide uses "versions" for major versions and "revisions" for minor versions.

### Deprecation vs. sunsetting

#### Deprecation

Deprecation marks a standard step in an API’s lifecycle. It applies to an entire API version or a part of it, such as a single operation, and indicates the availability of a better alternative, like a newer version or operation. 

> NB: Deprecation does not cause a breaking change. The Cisco Meraki Dashboard API currently uses version 1, which remains active and is not deprecated.

When Cisco Meraki deprecates an API version, it indicates that a superior operation is available and follows up with a sunset announcement and timelines when appropriate. Migrating to new operations is beneficial, particularly for large settings or specific use cases. 

Using a deprecated API version is not recommended. Developers should review their needs and explore available replacements, even if there is no sunset date.

#### Sunsetting

Sunsetting refers to the process of discontinuing support for either a specific API operation or an entire version after the deprecation period has ended. An operation or version is considered sunset once this period has concluded. It is important to note that sunsetting is a breaking change.

## Deprecated operations

In addition to the OAS, deprecated operations are documented in detail [here](deprecated-operations.md).
