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


# Lab 2 - Governance/Policy as Code 

**Block production toggling of feature flags outside the change control**

1. From the secondary menu, select **Project Settings** and select **Policies**

2. Select the **Policies** tab 

3. click **+ New Policy**, set the name to **Block Manual Toggle** and click **Apply**

4. Set the rego to the following and click **Save**

<!---->
	package feature_flags
	
	deny[msg] {
		# Match flags where the "Production" environment is on ...
		prod := input.flag.envProperties[_]
		prod.environment == "prod"
		prod.pipelineConfigured == false
		prod.variationMap[_].targets[_].identifier == "webinar"
		prod.variationMap[_].variation == "true"
		# Show a human-friendly error message
		msg := sprintf(`Flag '%s' cannot be enabled in "Production""`, [input.flag.name])
	}


5. Select the **Policy Sets** tab

6. Click **+ New Policy Set** and configure as follows

| Input                      | Value                     | Notes |
| -------------------------- | ------------------------- | ----- |
| Name                       |feature_flag_block_prod|       |
| Entity Type                |Feature Flag|       |
| Event Evaluation           |On Save|       |
| Policy Evaluation Criteria |                           |       |
| Policy to Evaluate         |Block Manual Toggle|       |

7. For the new policy set, toggle the **Enforced** button

**Test Policy**
1. From the left hand side menu navigate to the feature_flags
2. Disable/Enable and see what happens
3. Then navigate to targets, select the webinar target and try to display the show offer flag 

# Lab 3 - Feature Flag Pipeline
1. From the left hand side menu select pipelines
2) Click **+ Create a Pipeline**, enter the following values, then click **Start**

| Field                                  | Value            | Notes
| -------------------------------------- | ---------------- | ------------------------------------------------------------------------------------------ |
| Name                                   |feature_flag|                                                                                            |
| How do you want to setup your pipeline |Inline| This indicates that Harness (rather than Git) will be the source of truth for the pipeline |

3) Click **Add Stage** and select Feature Flag
4) Enter the following values and click on **Set Up Stage**

| Input           | Value           | Notes |
| --------------- | --------------- | ----- |
| Stage Name      |FF|       |


4) Click **Add Step** and from the Step library select **Flag Configuration**
| Input           | Value           | Notes |
| --------------- | --------------- | ----- |
| Step Name Name      |Toggle|       |

5) Set all other values as runtime inputs

6) From the left handside navigate to feature flags
7) Select the flag created earlier **webinarff**

8)  From the top navigation select **Flag Pipeline** and pick the pipeline created earlier
9)  Click on **Set as Flag Pipeline**

<img width="1653" height="955" alt="image" src="https://github.com/user-attachments/assets/24dd3eb1-1366-4e24-8f74-0c37a05b9eaa" />

Navigate to the target and try to change the flag status
Validate the pipeline execution 







