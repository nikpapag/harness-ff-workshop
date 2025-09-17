# Lab 1 - Feature Flags

### Summary: Build and deploy your first feature flag 

**Learning Objective(s):**

- Create a Feature Flag

- Create an SDK key

- Deploy application that uses Flag/SDK Key

- Toggle Feature Flag to enable/disable feature

**Steps**

![](https://lh7-us.googleusercontent.com/docsz/AD_4nXdxbh_5hgTG2CsE8Dp_5_BLB75OITfS-9xxW-xplPehdYbj38WMTloCOo4tbOAom9VRc65S99IB54w-TY7INiG6Bd8PMqvRs_EsTQHzKjCZTjnv8laP7XCEuf9_l3s8HV3UuxVsnTgzuZpkV6Fq-FVoqpHY5kSuQ3un7Xrssg?key=cRG2cvp_PHVW0KG2Gq6Y_A)

**Create the SDK Key**  

1. From the left hand side menu under Feature Flags,  select **environments**

2. From the list select the prod environment

3. Click **+ New SDK Key**, configure as follows and click **Create**

| Input    | Value      | Notes |
| -------- | ------     | ----- |
| Name     |sdk|       |
| Key Type |client|       |

4. Copy the secret to use later. Note that the key will be redacted once you leave the page.

5. From the left hand side menu select Project settings

6. From the resources available click on the **Variables** 

7. Modify the sdk variable and copy in the key

| Input | Value                               | Notes |
| ----- | ----------------------------------- | ----- |
| Name  |sdk|       |
| Value | _SDK Key copied from previous step_ |       |

4. Click **Save**

********

**Create the Flag**

1. From the left hand menu, go to **Feature Flags** → **Feature Flags**

2. Click **+ New Feature Flag,** configure as follows and click **Save and Close**.

| Input                         | Value      | Notes |
| ----------------------------- | --------------   | ----- |
| Type                          |Boolean|       |
| Name                          |webinarff|       |
| **Variation Settings**        |                  |       |
| Name (first one)              |Show Offer|       |
| Name (second one)             |Hide Offer|       |
| If the flag is Enabled, serve |Show Offer|       |

3. Enable the flag by clicking on the **Flag is Disabled** button and click **Save**


5. **Run** the pipeline created in previous steps

6. **Approve canary deployment** before progressing to the next step

**Change the Flag via the UI**

1. From the left hand menu in Harness, go to **Feature Flags** → **Target Management**

2. Select the target shown in the list. If target is not shown, create the target manually

| Input      | Value     | Notes |
| ---------- | --------- | ----- |
| Name       |webinar|       |
| Identifier |webinar|       |

3. Click **Add Flag**, toggle **webinarff**, set the variation to **Show Offer**, then click on **Add 1 Flags**

4. Note that your application now displays a special offer

5. For your target, set the variation to **Hide Offer** and click **Save Chances**

6. Note that your application now does NOT display the special offer




# Lab 8 - Governance/Policy as Code

### Summary: Create and apply policies as code in order to enable governance and promote self-service. In Lab 2 we saw how a user is impacted by policies in place, now is the time to create such policies

**Learning Objective(s):**

- Create a policy that evaluates when editing pipelines

- Create a policy that evaluates during pipeline execution

- Test policy enforcement

**Steps**
**Create a Policy to require Approvals**

1. From the secondary menu, select **Project Settings** and select **Governance Policies**

2. Click **Build a Sample Policy**

3. From the suggested list select **Pipeline - Approval**  and click on next

4. Click Next: Enforce Policy

5. Set the values according to the table  below and confirm

| Input            | Value        | Notes |
| ---------------- | ------------ | ----- |
| Trigger Event    |On Run|       |
| Failure Strategy |Error & exit|       |

**Test the Policy to require Approvals**

1. Open your pipeline

2. Try to run the pipeline and note that the failure due to lack of an approval stage

3. Open the pipeline in edit mode and navigate to the “**frontend**” stage

4. Before the rolling deployment step add **Harness Approval** step according to the table  below

| Input            | Value            | Notes |
| ---------------- | ---------------- | ----- |
| Step Name        |Approval|       |
| Type of Approval |Harness Approval|       |

5. Configure the Approval step as follows

| Input       | Value             | Notes |
| ----------- | ----------------- | ----- |
| Name        |Approval|       |
| User Groups |All Project Users|       |

6. Navigate to the “**backend**” stage
7. Repeat steps 4-5 to add an approval **before** the canary deployment block 
8. Click **Save** and note that the save succeeds without any policy failure


# Lab 9 - Governance/Policy as Code (Advanced)

**Create a Policy to block critical CVEs**

1. From the secondary menu, select **Project Settings** and select **Policies**

2. Select the **Policies** tab 

3. click **+ New Policy**, set the name to **Runtime OWASP CVEs** and click **Apply**

4. Set the rego to the following and click **Save**

<!---->

    package pipeline_environment
    deny[sprintf("Node OSS Can't contain any critical vulnerability '%d'", [input.NODE_OSS_CRITICAL_COUNT])] {  
       input.NODE_OSS_CRITICAL_COUNT != 0
    }

5. Select the **Policy Sets** tab

6. Click **+ New Policy Set** and configure as follows

| Input                      | Value                     | Notes |
| -------------------------- | ------------------------- | ----- |
| Name                       |Criticals Not Allowed|       |
| Entity Type                |Custom|       |
| Event Evaluation           |On Step|       |
| Policy Evaluation Criteria |                           |       |
| Policy to Evaluate         |Runtime OWASP CVEs|       |

7. For the new policy set, toggle the **Enforced** button

**Add Policy to Pipeline**

1. Open your pipeline

2. Go to an execution that already ran, and copy the CRITICAL output variable from the OWASP step like so:\
   ![](https://lh7-us.googleusercontent.com/docsz/AD_4nXfYQ7ba5Q_cQ9xy2AFVZ5Mt0iZPYbyQDmBonp0pBQA13Z_IUeYdK8gRSbddtf_V3bSRfbhKWDbRSUVJTx3BTCc_VmwLIWyWLkdh89nLh0sEBA6fqQxTy0NADZ0YPZwCirNycRVGUQACdItaBotovPs5Hg6CmRpQHk5ysgV6RUlhSbIbkNxmHAo?key=cRG2cvp_PHVW0KG2Gq6Y_A)

3. Select the **frontend** stage

4. Before the **Rollout Deployment** Step Group, add a **Policy** type step and configure as follow

| Input       | Value                                          | Notes                                                                                                                                                   |
| ----------- | ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name        |Policy - No Critical CVEs|                                                                                                                                                         |
| Entity Type |Custom|                                                                                                                                                         |
| Policy Set  |Criticals Not Allowed| Make sure to select the Project tab in order to see your Policy Set                                                                                     |
| Payload     |{"NODE\_OSS\_CRITICAL\_COUNT": _\<variable>_}| Set the field type to Expression, then replace _\<variable>_ with OWASP output variable CRITICAL. Go to a previous execution to copy the variable path. |

5. Save the pipeline and execute. Note that the pipeline fails at the policy evaluation step due to critical vulnerabilities being found by OWASP.
