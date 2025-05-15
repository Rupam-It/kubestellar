Testing the Chart this is a dry run
Navigate to the core chart directory:
```bash
cd kubestellar/core-chart
```
1. Update Dependencies
First, update the chart dependencies:
```bash
helm dependency update
```
2. Render Templates
Render the Helm templates to verify they process correctly:
```bash
helm template . > rendered.yaml
```
3. Verify the Output
Check if key components are properly rendered in the output:
Search for KubeFlexOperator
```bash
grep -i KubeFlexOperator rendered.yaml
```
Search for ArgoCD
```bash
grep -i argocd rendered.yaml
```
