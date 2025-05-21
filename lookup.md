# Using Helm Lookup Function in KubeStellar

## Overview

This document outlines how to use Helm's `lookup` function to simplify and improve the KubeStellar and KubeFlex installation process. 
## Benefits

- **Improved Security**: Removes dependency on kubectl images and minimizes RBAC requirements
- **Simplified User Experience**: Automates detection of existing components
- **Cleaner Code**: Eliminates complex bash scripts and init containers
- **Better Dependency Management**: Handles resource prerequisites more elegantly

## Implementation Areas

### 1. KubeFlex Namespace Creation

Replace pre-create hooks with simple lookup-based conditional rendering:

```yaml
{{- if not (lookup "v1" "Namespace" "" "kubeflex") }}
apiVersion: v1
kind: Namespace
metadata:
  name: kubeflex
{{- end }}
```

### 2. Automatic KubeFlex Operator Detection

Eliminate the need for users to manually set `kubeflex-operator.install`:

```yaml
{{- $kflexOperator := lookup "apps/v1" "Deployment" "kubeflex" "kubeflex-operator" }}
{{- if not $kflexOperator }}
  # Include kubeflex operator installation
{{- else }}
  # Skip kubeflex operator installation but use the existing one
{{- end }}
```

### 3. PostgreSQL Chart Integration

Handle the null value problem more elegantly:

```yaml
{{- $postgresql := lookup "apps/v1" "StatefulSet" .Release.Namespace "postgresql" }}
{{- if not $postgresql }}
postgresql:
  enabled: true
  # Other PostgreSQL configuration
{{- else }}
postgresql:
  enabled: false
externalPostgresql:
  host: postgresql.{{ .Release.Namespace }}.svc.cluster.local
{{- end }}
```

### 4. ITS/WDS Component Management

Simplify post-create hooks for InboundTrafficSystem and WorkloadDeploymentSystem:

```yaml
{{- $its := lookup "kubestellar.io/v1alpha1" "InboundTrafficSystem" "" "default-its" }}
{{- if not $its }}
  # Include ITS installation resources
{{- end }}

{{- $wds := lookup "kubestellar.io/v1alpha1" "WorkloadDeploymentSystem" "" "default-wds" }}
{{- if not $wds }}
  # Include WDS installation resources
{{- end }}
```

### 5. Control Plane Status Checks

Check for Control Plane existence and status (though waiting still requires an operator):

```yaml
{{- $cp := lookup "kubestellar.io/v1alpha1" "ControlPlane" "" .Release.Name }}
{{- if and $cp (eq (index $cp "status" "phase") "Ready") }}
  # Control plane is ready, proceed with dependent resources
{{- else }}
  # Either warn user or set up different configuration
  {{- fail "Control Plane must be ready before installing this chart" }}
{{- end }}
```

### 6. Dependency Management

Create reusable templates for dependency handling:

```yaml
{{- define "kubestellar.installDependencies" -}}
{{- $kflex := lookup "apps/v1" "Deployment" "kubeflex" "kubeflex-operator" }}
{{- if not $kflex }}
  {{- .Values.kubeflex | set . "installKubeFlex" true -}}
{{- else }}
  {{- false | set . "installKubeFlex" -}}
{{- end }}
{{- end -}}
```

### 7. Prerequisite Validation

Fail with helpful error messages when dependencies aren't met:

```yaml
{{- $its := lookup "kubestellar.io/v1alpha1" "InboundTrafficSystem" "" .Values.itsName }}
{{- if not $its }}
  {{- fail (printf "Required ITS %s not found. Please install it first." .Values.itsName) }}
{{- end }}
```
