# How to update template

`template-update` branch is working branch for template updates.

```pwsh
chmod +x ./eng/scripts/Build-Templates.ps1
./eng/scripts/Build-Templates.ps1
```

## diff between infra code

azd-starter-bicep vs todo-python-mongo-aca. `app` and main.bicep are different.

```sh
$ diff -qr azd-starter-bicep/generated/infra todo-python-mongo-aca/generated/infra
Only in todo-python-mongo-aca/generated/infra: app
Files azd-starter-bicep/generated/infra/main.bicep and todo-python-mongo-aca/generated/infra/main.bicep differ
Files azd-starter-bicep/generated/infra/main.parameters.json and todo-python-mongo-aca/generated/infra/main.parameters.json differ
```

todo-java-mongo-aca and todo-python-mongo-aca are the same.

```sh
$ diff -qr todo-java-mongo-aca/generated/infra todo-python-mongo-aca/generated/infra
```

todo-python-mongo-aca's `infra/main.bicep` is copy from `templates/todo/projects/python-mongo-aca/.repo/bicep/infra/main.bicep`.

## Where do the files in the app directory come from?

`todo-python-mongo-aca/generated/infra/app`  has following files.

```sh
# azure-dev/.output/todo-python-mongo-aca/generated/infra
$ tree ./app
./app
├── api.bicep
├── apim-api-policy.xml
├── apim-api.bicep
├── db.bicep
└── web.bicep
```

Each file in the generated app directory `.output/todo-python-mongo-aca/generated/infra/app/` is copied from `templates/todo/common/infra/bicep/` and modify relative path.
Let's check it out.

```sh
# base path from cloned azure-dev root.
$ export GENERATED=.output/todo-python-mongo-aca/generated/infra/
$ export TEMPLATES=templates/todo/common/infra/bicep/
```

```sh
# diff between api
$ diff -u $GENERATED/app/api.bicep $TEMPLATES/app/api-container-app.bicep
```

```diff
--- .output/todo-python-mongo-aca/generated/infra//app/api.bicep        2024-02-17 14:37:05.106048545 +0900
+++ templates/todo/common/infra/bicep//app/api-container-app.bicep      2024-02-17 14:36:47.036055414 +0900
@@ -17,7 +17,7 @@
 }

 // Give the API access to KeyVault
-module apiKeyVaultAccess '../core/security/keyvault-access.bicep' = {
+module apiKeyVaultAccess '../../../../../common/infra/bicep/core/security/keyvault-access.bicep' = {
   name: 'api-keyvault-access'
   params: {
     keyVaultName: keyVaultName
@@ -25,7 +25,7 @@
   }
 }

-module app '../core/host/container-app-upsert.bicep' = {
+module app '../../../../../common/infra/bicep/core/host/container-app-upsert.bicep' = {
   name: '${serviceName}-container-app'
   dependsOn: [ apiKeyVaultAccess ]
   params: {

```sh
# diff between db
$ diff -u $GENERATED/app/db.bicep $TEMPLATES/app/cosmos-mongo-db.bicep
```

```diff
--- .output/todo-python-mongo-aca/generated/infra//app/db.bicep 2024-02-17 14:37:05.106048545 +0900
+++ templates/todo/common/infra/bicep//app/cosmos-mongo-db.bicep        2024-02-17 14:36:47.036055414 +0900
@@ -23,7 +23,7 @@
 var defaultDatabaseName = 'Todo'
 var actualDatabaseName = !empty(databaseName) ? databaseName : defaultDatabaseName

-module cosmos '../core/database/cosmos/mongo/cosmos-mongo-db.bicep' = {
+module cosmos '../../../../../common/infra/bicep/core/database/cosmos/mongo/cosmos-mongo-db.bicep' = {
   name: 'cosmos-mongo'
   params: {
     accountName: accountName    
```

```sh
# diff between web
$ diff -u $GENERATED/app/web.bicep $TEMPLATES/app/web-container-app.bicep

```diff
--- .output/todo-python-mongo-aca/generated/infra//app/web.bicep        2024-02-17 14:37:05.106048545 +0900
+++ templates/todo/common/infra/bicep//app/web-container-app.bicep      2024-02-17 14:36:47.036055414 +0900
@@ -15,7 +15,7 @@
   location: location
 }

-module app '../core/host/container-app-upsert.bicep' = {
+module app '../../../../../common/infra/bicep/core/host/container-app-upsert.bicep' = {
   name: '${serviceName}-container-app'
   params: {
     name: name
```



