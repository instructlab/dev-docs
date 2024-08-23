# `ilab` Config.yaml Versioning

## Problem Statement

Currently the `ilab` CLI's configuration file, `config.yaml`, does not contain a version.

The config file in `ilab` version `0.17` cannot be used in future versions of `ilab` due to new mandatory fields and changes to existing fields.

As the configuration file grows, changes, and becomes more complex, versioning is necessary in order to support forwards and backwards compatibility.

## Proposal

The `ilab` configuration file has a new `version` field at the top level of the config.

The value of `version` should start at `1.0.0`.

## Future Considerations

As future development of `ilab` occurs, ramifications and tooling for the versioning the configuration file will need to be determined for users and developers.

Such considerations include but are not limited to:

- Rules for incrementing the version number
- Tooling to support forward compatibility of configuration files
- Limits for the forward and backward compatibility of configuration files
