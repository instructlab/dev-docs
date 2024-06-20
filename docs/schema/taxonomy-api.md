# Central API for taxonomy reading and validation

Current there are multiple places where the taxonomy `qna.yaml` files are read, parsed, and validated. There is a `check_yaml.py` script in the `taxonomy` repository and there are methods in the `instructlab` repository in the `src/instructlab/util.py` file.

The methods in `utils` are used by both the `ilab taxonomy diff` command as well as in the SDG code which has been moved to the `sdg` repository. This arrangement results in a circular dependency between the `instructlab` package to access the SDG code and from the SDG code in the `instructlab-sdg` package to access the `utils` methods to read and validate the taxonomy files.

## Use instructlab-schema package for the central API

We now have an `instructlab-schema` package on PyPI which holds the JSON schema files for the taxonomy `qna.yaml` files. This is now used by `instructlab` to access these schema files for taxonomy file validation.

We should relocate the taxonomy reading and validation code from `instructlab` to `instructlab-schema`. This will provide for a central place near to the JSON schema it uses for a shared API for reading, parsing, and validating taxonomy `qna.yaml` files.

Then we can modify the `instructlab` and `instructlab-sdg` packages to depend upon the `instructlab-schema` package for these APIs which will remove a circular dependency. We can also use these APIs in the taxonomy repositories `check_yaml.py` script as well.
