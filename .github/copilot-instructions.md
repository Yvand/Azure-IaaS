# Repository instructions

## Build and validation

- Build the resource-group template and regenerate its checked-in ARM JSON: `az bicep build --file main.bicep --outfile azuredeploy.json`. Keep `azuredeploy.json` in sync with `main.bicep`.
- The ARM Template Test Toolkit (ARM-TTK) suite runs in `.github/workflows/run-arm-ttk.yml` on pull requests to `master` and by manual dispatch. Locally, with ARM-TTK installed and `ARMTTK_PATH` set, run the full suite with `Import-Module "$env:ARMTTK_PATH/arm-ttk.psd1"; Test-AzTemplate -TemplatePath . -Skip "artifacts-parameter"`.
- To run one ARM-TTK test, use `Import-Module "$env:ARMTTK_PATH/arm-ttk.psd1"; Test-AzTemplate -TemplatePath . -Test <test-name>`.
- There is no separate unit-test or lint command configured. The Bicep build performs compilation and diagnostics; the GitHub workflow runs ARM-TTK.

## Architecture

- `main.bicep` is the resource-group-scope entry point. It defines the SharePoint version and configuration options, computes shared environment/role settings, and orchestrates the network, base virtual machines, optional SharePoint front ends, optional Bastion, and optional Azure Firewall.
- `virtualNetwork.bicep`, `virtualMachine.bicep`, `bastion.bicep`, and `firewall.bicep` are focused modules. They provision resources through pinned Azure Verified Module (AVM) versions where applicable, while `virtualMachine.bicep` also configures the VM run command and DSC extension.
- The virtual machines are the domain controller, SQL Server, and SharePoint main server, plus zero to four optional SharePoint front ends. Their role-specific DSC packages and configuration scripts are downloaded from the `SharePointInfraDsc` release URL configured by `_artifactsLocation`; the DSC implementation is external to this repository.
- `main.bicepparam` is a local deployment example. `azuredeploy.json` is the compiled ARM-template counterpart of `main.bicep` and is used by the Azure Quickstart deployment links and metadata.
- `DeployTemplate.ps1` and `.vscode/tasks.json` provide Azure PowerShell/Azure CLI deployment entry points. The GitHub Action in `action-armttk` wraps ARM-TTK for the CI workflow.

## Repository-specific conventions

- Keep deployment parameters, validation constraints, defaults, environment calculations, and role-specific DSC argument wiring in `main.bicep`; keep reusable resource implementation in its corresponding module.
- When changing a root parameter or SharePoint configuration option, follow it through the derived DSC settings and deployment modules, and update the usage documentation in `README.md` when user-facing behavior changes.
- Keep AVM references explicitly version-pinned in Bicep module declarations.
- Treat `adminPassword`, `otherAccountsPassword`, and artifact access tokens as secrets: preserve secure parameter annotations and pass credentials to DSC through protected settings, not ordinary settings.
- The supported SharePoint configuration levels and feature names are defined by the allowed values in `main.bicep`; preserve these names and the mapping from configuration levels to selected features when changing configuration behavior.
