==============================================================================================================
                                        Understand Templating Engine
==============================================================================================================

Helm uses:

- Go template engine
- Sprig functions

Example:

metadata:
  name: {{ include "my-app.fullname" . }}

. → root context
.Values → values.yaml
.Chart → chart metadata
.Release → release metadata