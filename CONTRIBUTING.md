# Contributing

All members of the Open Source Community are welcome to contribute to this project. Contributions can be received through a variety of forms including source code contributions, reviews and issue creation. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Contributors and Committers

Individuals who are interested in directly affecting repository content may request the privilege in becoming a committer. After an individual has made number of high quality contributions that demonstrate that they understand how the project works, they can request commit access or be nominated by another member of the community. All requests for committer status will be reviewed and decided by existing committers transparently.

## Prerequisites

When running from source (that is, debugging in the Extension Development Host), you must install dependencies first.  There are two sets of dependencies: node module dependencies and extension dependencies.

1. **Install node modules:** In the integrated terminal, run `npm install`.
2. **Install extension dependencies:** In the VS Code Extensions tab, search for RedHat YAML and install that extension.

(Note that end users do *not* need to do these steps.  VS Code extension packaging and installation takes care of them.  But running from source bypasses this process so we have to do them by hand.)

If you get the error `Extension 'ms-kubernetes-tools.vscode-kubernetes-tools' failed to activate` (usually followed by `command 'extension.some-command-id' not found`) then it probably means you're missing an extension dependency.

## The 'Create Cluster' and 'Add Existing Cluster' Commands

The 'Create Cluster' and 'Add Existing Cluster' commands rely on provider-specific tools or APIs for interacting with the cloud or cluster.  We have a basic extension point for implementing new cloud providers.  **This is currently experimental and we welcome feedback.**

To implement a provider for the 'Create Cluster' and/or 'Add Existing Cluster' commands, your extension needs to do the following:

* Set itself for automatic activation (the `*` activation event).  We know this isn't great and would love to discuss a better approach.
* Get the VS Code Kubernetes extension's API object via the `vscode.extensions.getExtension<T>` and `Extension<T>.activate()` functions.
* The API object has a field named `clusterProviderRegistry`.  This is an object you can use to register the type(s) of cluster you support.  Call the registry's `register` method, passing an object with the following fields:
  * `id` (string): an identifier for the type of cluster.  This needs to be unique across all providers, as we use it to dispatch the user's selected cluster type to the appropriate handler.  E.g. for Azure Kubernetes Service we use `aks`.
  * `displayName` (string): the display name for the type of cluster.  E.g. for Azure Kubernetes Service we use (wait for it) `Azure Kubernetes Service`.
  * `supportedActions` (string[]): an array of strings specifying which commands you support for this cluster type.  The supported commands are `configure` (for Add Existing Cluster) and `create` (for Create Cluster) - any other strings in the array will be ignored.
  * `next` (method): the function to be invoked when the user chooses your cluster type, or moves to the next page in your cluster wizard.
* When the user choose 'Create Cluster' or 'Add Existing Cluster', the extension displays a list of all registered cluster types supporting the appropriate command (using the `displayName` attribute and filtering using the `supportedActions` attribute).
* When the user selects a cluster type, the extension calls the `next` method, passing:
  * the `Wizard` object representing the VS Code webview where the UI is being hosted
  * an action string - `configure` or `create` according to which command the user is executing
  * an object representing the data from the webview - in the first call this will have only a `clusterType` attribute
* Your `next` implementation must call the Wizard object's `showPage` method, passing the HTML you would like to display.  Your HTML must obey the following conventions:
  * It will be hosted in a `div` element, so should not include e.g. `head` or `body` tags.
  * It must contain a form named `form`, which must at minimum contain a hidden input named `clusterType` whose value your provider's cluster type `id`.  Any other user inputs that you want to be able to process must also be in this form.
  * To proceed to the next page (e.g. from a `Next >` link), it must call the `onNext()` function which is provided by the wizard hosting page.
* When your HTML calls `onNext()`, your `next` method will be called again, and this time the message will contain a map of the form fields.

For an example of this, see the `components/clusterprovider/azure` directory.  If you have any questions or run into any problems, please post an issue - we'll be very happy to help.
