
================= The correct and simple way to switch from your Production subscription to your Test subscription in Azure CLI (Cloud Shell or local). ==================================

-----------------------------------------------------------------------------------------------------------
✔ Step 1 — List all subscriptions

az account list --output table

You will get output like:

Name        CloudName    SubscriptionId                        TenantId                              State    IsDefault
----------  -----------  ------------------------------------  ------------------------------------  -------  -----------
Production  AzureCloud   5a6c11db-2cbd-4313-ad43-3e29b4017a6c  7faa8f70-7da4-4fa9-a24d-9bbd6682a962  Enabled  True
Test        AzureCloud   52f167e5-dfca-4d77-a744-e6c7bc1a3235  7faa8f70-7da4-4fa9-a24d-9bbd6682a962  Enabled  False

Copy the SubscriptionId of your Test subscription.
------------------------------------------------------------------------------------------------------------

-----------------------------------------------------------------------------------------------------------
✔ Step 2 — Set the Test subscription as active

Replace <TEST_SUBSCRIPTION_ID> with your actual ID:

az account set --subscription "52f167e5-dfca-4d77-a744-e6c7bc1a3235"

------------------------------------------------------------------------------------------------------------

✔ Step 3 — Verify your CLI is now using the Test subscription

az account show --output json

You should see:

"name": "Test",
"isDefault": true,
"id": "<TEST_SUBSCRIPTION_ID>"

-----------------------------------------------------------------------------------------------------------


===================================== interactive login that’s more reliable for MFA ================================

Try an interactive login that’s more reliable for MFA / conditional access

If your organization uses MFA or conditional access, run:

az login --use-device-code

This prints a short URL and code you open in a browser and complete the MFA. It often avoids GUI popup issues.

===============================================================================================================


az account show --output json
az account list --output table

az account set --subscription "<TEST_SUBSCRIPTION_ID>"
az account show --output table

============================================================================================================

tejas [ ~ ]$ az ad group show --group aks-dev-team --query id -o tsv
bfb55ed7-fc0c-44dc-84b9-e263cbe30c26