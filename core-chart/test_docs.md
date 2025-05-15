Here’s a `README.md` file that explains what you've done in the `kubesteller/core-chart` directory, including Helm commands and how to search within the generated YAML file:

````markdown
# Kubesteller Core Chart Setup

This guide outlines the steps to render Kubernetes manifests using Helm and locate specific resources such as `KubeFlexOperator` and `ArgoCD` in the output.

## Steps Performed

Inside the `kubesteller/core-chart` directory, the following steps were executed:

1. **Update Chart Dependencies**  
   This command fetches and updates the dependencies listed in `Chart.yaml`:
   ```bash
   helm dependency update
````

2. **Render Helm Template**
   This command renders the Kubernetes manifests into a single YAML file:

   ```bash
   helm template . >rendered.yaml
   ```

3. **Copy Rendered File (Optional)**
   To move the rendered YAML file to your current directory:

   ```bash
   cp rendered.yaml .
   ```

## Searching for Specific Resources

Once you have the `rendered.yaml` file, you can search for specific components or custom resources using `grep`.

### Search for KubeFlexOperator

```bash
grep -i Kubeflexoperator rendered.yaml
```

### Search for ArgoCD

```bash
grep -i argocd rendered.yaml
```


