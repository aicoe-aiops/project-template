# Automated Argo workflows

If you'd like to automate your Jupyter notebooks using Argo, please use these kustomize manifests. If you follow the steps bellow, your application is fully set and ready to be deployed via Argo CD.

For a detailed guide on how to adjust your notebooks etc, please consult documentation in the [OpenShift SME project](https://github.com/aicoe-aiops/openshift-sme-mailing-list-analysis/blob/master/argo/README.md)

1. Replace all `<VARIABLE>` mentions with your project name, respective url or any fitting value
2. Define your automation run structure in the `templates` section of [`cron-workflow.yaml`](./cron-workflow.yml)
3. Set up `sops`:

   1. Install `go` from your distribution repository
   2. Setup `GOPATH`

      ```bash
      echo 'export GOPATH="$HOME/.go"' >> ~/.bashrc
      echo 'export PATH="${GOPATH//://bin:}/bin:$PATH"' >> ~/.bashrc
      source  ~/.bashrc
      ```

   3. Install `sops` from your distribution repository if possible or use [sops GitHub release binaries](https://github.com/mozilla/sops#stable-release)

   4. Import AICoE-SRE's public key [EFDB9AFBD18936D9AB6B2EECBD2C73FF891FBC7E](https://keyserver.ubuntu.com/pks/lookup?op=get&search=0xefdb9afbd18936d9ab6b2eecbd2c73ff891fbc7e):

      ```bash
      gpg --keyserver keyserver.ubuntu.com --recv EFDB9AFBD18936D9AB6B2EECBD2C73FF891FBC7E
      ```

   5. If you'd like to be able to build the manifest on your own as well, please list your GPG key in the [`.sops.yaml` file](.sops.yaml), `pgp` section. With your key present there, you can later generate the full manifests using `kustomize` (with `ksops` installed, please follow ksops [guide](https://github.com/viaduct-ai/kustomize-sops#0-verify-requirements).

4. Create a secret and encrypt it with `sops`:

   ```bash
   # If you're not already in the `manifest` folder, cd here
   cd manifests
   # Mind that `SECRET_NAME` must match the `SECRET_NAME` used in `cron-workflow.yaml`
   oc create secret generic <SECRET_NAME> \
     --from-literal=path=<BASE_PATH_WITHIN_CEPH_BUCKET> \
     --from-literal=access-key-id=<AWS_ACCESS_KEY_ID> \
     --from-literal=secret-access-key=<AWS_SECRET_ACCESS_KEY> \
     --dry-run -o yaml |
   sops --input-type=yaml --output-type=yaml -e /dev/stdin > ceph-creds.yaml
   ```
