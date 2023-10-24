# Example structure of a Daml project
It is typically desirable to structure a Daml project in a way that it would be compiled into multiple .dar files. This allows applications to be composable. Once you finished a module and compilted it into a .dar file, you can then develop further modules that utilize templates provided by the first module, thus extending the functionality of the application. Splitting the overall app into multiple .dar files also allows to upgrade parts of the app at different frequency.  
However, composing an application from multiple .dars presents a problem for editing the code in VS Code IDE. Just importing modules from other .dar files, where the existing .dars are specified as a dependency or data dependency, does not allow to edit imported modules and have these edits immediately reflected in the importing modules. This is because in this setup the imports are from the compiled .dar files rather than from the source .daml files.  

This is an example of how to structure Daml projects to be able to both edit code in VS Code and to compile the project into multiple .dar files.
The project structure consists of a folder named "src", which contains the source code of the project organized into whatever folder structure is convenient for the source code. In our example the folder structure under the "src" folder separates the modules that should be compiled into different .dar files.
The project structure also includes a folder named "package". This folder provides organization for the package. It contains a separate daml.yaml file for each .dar file we want to create. The "package" folder does not contain any source code. Instead it contains symlinks that point to the source code files or folders underneath the "src" folder mentioned above.

Follow the instructions below to build and test this app.

## Build the superpackage
Using a command shell (Terminal or Command Prompt) from the root directory of the project run
```
daml build
```
This will build a single .dar file in the root of the project from all source code modules. This .dar file is not used to deploy the application. Its sole purpose is to allow editing of the source files in VS Code.

## Launch VS Code
Using a command shell (Terminal or Command Prompt) from the root directory of the project run
```
daml studio
```
This will launch VS Code application, where you can edit the source code under the "src" folder. Note that any changes you make to say Asset.daml module under src/Asset are immediately propagated to other modules that import Asset.daml, e.g. src/Account/Account.daml or src/Tests/Account/Test.daml

## Build deployment package

Open up a command shell and from the project root folder, run the following commands:
```
cd package/User
daml build -o user.dar

cd ../Asset
daml build -o asset.dar

cd ../Account
daml build -o account.dar
daml build -o setup.dar

cd ../Tests
daml build -o tests.dar
```
This will generate four .dar files: asset.dar, account.dar, user.dar, tests.dar in their respective folders. The asset.dar, account.dar, user.dar can then be deployed on the ledger. Once deployed on the ledger, these files provide the backend for the Wallet Daml Sample App.

## Run the test scripts for the Wallet App

Open up a command shell and from the project root folder, run the following command:
```
cd package/tests
daml test --all --show-coverage
```

The expected output from the above is
```
Test Summary

Tests/Account/Test.daml:transfer_privacy_test: ok, 0 active contracts, 0 transactions.
Tests/Account/Test.daml:setupParties: ok, 0 active contracts, 0 transactions.
Tests/Account/Test.daml:accountBalance_test: ok, 3 active contracts, 3 transactions.
Tests/Account/Test.daml:dupe_account_test: ok, 2 active contracts, 3 transactions.
Tests/Account/Test.daml:account_recipient_mismatch_test: ok, 5 active contracts, 6 transactions.
Tests/Account/Test.daml:account_airdrop_test: ok, 5 active contracts, 8 transactions.
Tests/Account/Test.daml:dupe_account_invite_test: ok, 3 active contracts, 5 transactions.
Tests/Account/Test.daml:account_reshare_test: ok, 5 active contracts, 8 transactions.
Tests/Account/Test.daml:transfer_test: ok, 4 active contracts, 8 transactions.
Tests/Account/Test.daml:trade_test: ok, 8 active contracts, 12 transactions.
Tests/Asset/Test.daml:setupParties: ok, 0 active contracts, 0 transactions.
Tests/Asset/Test.daml:asset_tests: ok, 1 active contracts, 4 transactions.
wallet-account:createPublic: ok, 0 active contracts, 0 transactions.
wallet-account:setup: ok, 3 active contracts, 3 transactions.
test coverage: templates 75%, choices 36%
templates never created:
wallet-account:Account.Account:AirdropRequest
wallet-account:Account.Account:AssetHoldingAccountCloseProposal
wallet-account:Account.Account:AssetHoldingAccountRequest
choices never executed:
wallet-user:User.User:Alias:Archive
wallet-user:User.User:Alias:Change
wallet-user:User.User:User:Archive
wallet-account:Account.Account:AirdropRequest:AirdropRequest_Accept
wallet-account:Account.Account:AirdropRequest:Archive
wallet-account:Account.Account:AssetHoldingAccount:Archive
wallet-account:Account.Account:AssetHoldingAccount:Close_Account
wallet-account:Account.Account:AssetHoldingAccountCloseProposal:Archive
wallet-account:Account.Account:AssetHoldingAccountCloseProposal:AssetHoldingAccountCloseProposal_Accept
wallet-account:Account.Account:AssetHoldingAccountCloseProposal:AssetHoldingAccountCloseProposal_Reject
wallet-account:Account.Account:AssetHoldingAccountProposal:Archive
wallet-account:Account.Account:AssetHoldingAccountProposal:AssetHoldingAccountProposal_Reject
wallet-account:Account.Account:AssetHoldingAccountRequest:Archive
wallet-account:Account.Account:AssetHoldingAccountRequest:AssetHoldingAccountRequest_Accept
wallet-account:Account.Account:AssetInSwap:Cancel_Swap
wallet-account:Account.Account:AssetInSwap:Reject_Swap
wallet-account:Account.Account:Trade:Archive
wallet-account:Account.Account:Trade:Trade_Cancel
wallet-account:Account.Account:Trade:Trade_Reject
wallet-account:Account.Account:TransferPreApproval:Archive
wallet-account:Account.Account:TransferPreApproval:TransferPreApproval_Cancel
wallet-account:Account.Account:TransferPreApproval:TransferPreApproval_Reject
wallet-asset:Asset.Asset:AssetTransfer:Archive
wallet-asset:Asset.Asset:AssetTransfer:Cancel_Transfer
wallet-asset:Asset.Asset:AssetTransfer:Reject_Transfer
```
