# DevFileCatalogue

This is a demo of how DevFileRegistry can be deployed without building registry container.

## Deploy

```bash
oc project <your project with DevSpaces operator running>
oc apply -k .
```

## DevSpaces Configuration

Update Deployment configuration to work with external registy

```yaml
spec:
  components:
    cheServer:
      debug: false
      logLevel: INFO
    dashboard:
      logLevel: ERROR
    devWorkspace: {} 
    devfileRegistry:  # <- Add this section below
      externalDevfileRegistries:   
        - url: 'http://registry:8080'
    imagePuller:
      enable: false
      spec: {}
    metrics:
      enable: true
    pluginRegistry: {}
```

## Customization

Whole idea of this repo was to demonstrate how to connect DevWorkspaces to additional networks on OCP. To add extra interface to DevSpaces, modify these sections in the `configmap.yaml`

```yaml
    apiVersion: workspace.devfile.io/v1alpha2
    kind: DevWorkspace
    metadata:
      name: ansible-demo
      annotations:
        che.eclipse.org/devfile: |
          schemaVersion: 2.2.2
          metadata:
            name: ansible-demo
          components:
            - name: tooling-container
              attributes:
                che-theia.eclipse.org/vscode-extensions:
                  - 'relative:extension/resources/github_com/microsoft/vscode-python/releases/download/2020_7_94776/ms-python-release.vsix'
                che-theia.eclipse.org/vscode-preferences:
                  python.globalModuleInstallation: true
                # This section connects DevSpaces container to additional network
                pod-overrides: 
                  metadata: 
                    annotations:
                        k8s.v1.cni.cncf.io/networks: |
                          [
                            {
                              "name": "eve-l2-network",
                              "ips": ["192.168.100.101/24"]
                            }
                          ]   
.... 
    spec:
      started: true
      routingClass: che
      template:
        components:
          - name: tooling-container
            attributes:
              che-theia.eclipse.org/vscode-extensions:
                - 'relative:extension/resources/github_com/microsoft/vscode-python/releases/download/2020_7_94776/ms-python-release.vsix'
              che-theia.eclipse.org/vscode-preferences:
                python.globalModuleInstallation: true
                # This section connects DevSpaces container to additional
                pod-overrides:
                metadata:
                  annotations:
                      k8s.v1.cni.cncf.io/networks: |
                        [
                          {
                            "name": "eve-l2-network",
                            "ips": ["192.168.100.101/24"]
                          }
                        ]   

```


Happy ❤️ Developing 🧑‍💻