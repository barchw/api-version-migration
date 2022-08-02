# Migration to new API Rule version guide

## Implementing new version and conversion webhook

1. Create new APIRule **v1beta1** version with:

    ```sh
    kubebuilder create api --group gateway  --version v1beta1 --kind APIRule
    ```

2. Introduce api changes

3. Make sure that **v1beta1** api version is set as storage type

   ```go
   // In file api/v1beta1/apiRule_types.go

   // +kubebuilder:storageversion
   type APIRule struct {
   ```

4. Implement [hub and spoke](https://book.kubebuilder.io/multiversion-tutorial/conversion-concepts.html) that implements
`func (src *APIRule) ConvertTo(dstRaw conversion.Hub) error` and `func (dst *APIRule) ConvertFrom(srcRaw conversion.Hub) error`

   > Sidenote: If breaking changes are introduced in new api version, the `ConvertFrom` should throw an error if the conversion is not possible. However the conversion MUST be done if possible. If the `ConvertFrom` would just throw an error, creation of the resource in older version will fail (even though we are implementing NEW -> OLD conversion)

   > Other than that, ConvertFrom will not block any feature creation

5. Create conversion webhook

   ```sh
   kubebuilder create webhook --group gateway --version v1beta1 --kind APIRule --conversion
   ```

6. Set up conversion versions in config/crd/patches/webhook_in_apirules.yaml

7. Enable "patches/webhook_in_apirules.yaml" and "patches/cainjection_in_apirules.yaml" in config/crd/kustomization.yaml

8. Uncomment lines under [WEBHOOK], "../certmanager" and vars at the end of file in config/default/kustomization.yaml

9. Comment out "- manifests.yaml" in config/webhook/kustomization.yaml

10. ```make generate manifests install```

11. Build new controller version and deploy (make sure that certs are injected, e. g. [cert-manager](https://github.com/cert-manager/cert-manager/releases/download/v1.9.0/cert-manager.yaml))

12. Conversion should now be in place, the change should be visible with

    ```sh
    kubectl describe apirules.v1beta1.gateway.kyma-project.io
    ```
## External references
Based on: https://book.kubebuilder.io/multiversion-tutorial/tutorial.html

