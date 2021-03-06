# Hopsworks API Key

In order for the Databricks cluster to be able to communicate with the Hopsworks Feature Store, the clients running on Databricks need to be able to access a Hopsworks API Key.

## Generating an API Key

In Hopsworks, click on your *username* in the top-right corner and select *Settings* to open the user settings. Select *Api keys*. Give the key a name and select the job, featurestore and project scopes before creating the key. Copy the key into your clipboard for the next step.

!!! success "Scopes"
    The created API-Key should at least have the following scopes:

    1. featurestore
    2. project
    3. job

<p align="center">
  <figure>
    <img src="../../../assets/images/api-key.png" alt="Generating an API Key on Hopsworks">
    <figcaption>API-Keys can be generated in the User Settings on Hopsworks</figcaption>
  </figure>
</p>

!!! info
    You are only ably to retrieve the API Key once. If you miss to copy it to your clipboard, delete it again and create a new one.

## Quickstart API Key File

!!! hint "Save API Key as File"
    To get started quickly, without saving the Hopsworks API in a secret storage, you can simply create a file with the previously created Hopsworks API Key and place it on the environment from which you wish to connect to the Hopsworks Feature Store. That is either save it on the Databricks File System (DBFS) or in your Databricks workspace.

    You can then connect by simply passing the path to the key file when instantiating a connection:
    ```python hl_lines="6"
        import hsfs
        conn = hsfs.connection(
            'my_instance',                      # DNS of your Feature Store instance
            443,                                # Port to reach your Hopsworks instance, defaults to 443
            'my_project',                       # Name of your Hopsworks Feature Store project
            api_key_file='featurestore.key',    # The file containing the API key generated above
            hostname_verification=True)         # Disable for self-signed certificates
        )
        fs = conn.get_feature_store()           # Get the project's default feature store
    ```

## Storing the API Key

### AWS

#### Option 1: Using the AWS Systems Manager Parameter Store

**Storing the API Key in the AWS Systems Manager Parameter Store**

In the AWS Management Console, ensure that your active region is the region you use for Databricks.
Go to the *AWS Systems Manager* choose *Parameter Store* and select *Create Parameter*.
As name enter `/hopsworks/role/[MY_DATABRICKS_ROLE]/type/api-key` replacing `[MY_DATABRICKS_ROLE]` with the AWS role used by the Databricks cluster that should access the Feature Store. Select *Secure String* as type and create the parameter.

<p align="center">
  <figure>
    <a  href="../../../assets/images/databricks/aws/databricks_parameter_store.png">
      <img src="../../../assets/images/databricks/aws/databricks_parameter_store.png" alt="Storing the Feature Store API Key in the Parameter Store">
    </a>
    <figcaption>Storing the Feature Store API Key in the Parameter Store</figcaption>
  </figure>
</p>

**Granting access to the secret to the Databricks notebook role**

In the AWS Management Console, go to *IAM*, select *Roles* and then the role that is used when creating Databricks clusters.
Select *Add inline policy*. Choose *Systems Manager* as service, expand the *Read* access level and check *GetParameter*.
Expand Resources and select *Add ARN*.
Enter the region of the *Systems Manager* as well as the name of the parameter **WITHOUT the leading slash** e.g. *hopsworks/role/[MY_DATABRICKS_ROLE]/type/api-key* and click *Add*.
Click on *Review*, give the policy a name und click on *Create policy*.

<p align="center">
  <figure>
    <a  href="../../../assets/images/databricks/aws/databricks_parameter_store_policy.png">
      <img src="../../../assets/images/databricks/aws/databricks_parameter_store_policy.png" alt="Configuring the access policy for the Parameter Store">
    </a>
    <figcaption>Configuring the access policy for the Parameter Store</figcaption>
  </figure>
</p>

#### Option 2: Using the AWS Secrets Manager

**Storing the API Key in the AWS Secrets Manager**

In the AWS management console ensure that your active region is the region you use for Databricks.
Go to the *AWS Secrets Manager* and select *Store new secret*. Select *Other type of secrets* and add *api-key*
as the key and paste the API key created in the previous step as the value. Click next.

<p align="center">
  <figure>
    <a  href="../../../assets/images/databricks/aws/databricks_secrets_manager_step_1.png">
      <img src="../../../assets/images/databricks/aws/databricks_secrets_manager_step_1.png" alt="Storing a Feature Store API Key in the Secrets Manager Step 1">
    </a>
    <figcaption>Storing a Feature Store API Key in the Secrets Manager Step 1</figcaption>
  </figure>
</p>

As secret name, enter *hopsworks/role/[MY_DATABRICKS_ROLE]* replacing [MY_DATABRICKS_ROLE] with the AWS role used
by the Databricks instance that should access the Feature Store. Select next twice and finally store the secret.
Then click on the secret in the secrets list and take note of the *Secret ARN*.

<p align="center">
  <figure>
    <a  href="../../../assets/images/databricks/aws/databricks_secrets_manager_step_2.png">
      <img src="../../../assets/images/databricks/aws/databricks_secrets_manager_step_2.png" alt="Storing a Feature Store API Key in the Secrets Manager Step 2">
    </a>
    <figcaption>Storing a Feature Store API Key in the Secrets Manager Step 2</figcaption>
  </figure>
</p>

**Granting access to the secret to the Databricks notebook role**

In the AWS Management Console, go to *IAM*, select *Roles* and then the role that is used when creating Databricks clusters.
Select *Add inline policy*. Choose *Secrets Manager* as service, expand the *Read* access level and check *GetSecretValue*.
Expand Resources and select *Add ARN*. Paste the ARN of the secret created in the previous step.
Click on *Review*, give the policy a name und click on *Create policy*.

<p align="center">
  <figure>
    <a  href="../../../assets/images/databricks/aws/databricks_secrets_manager_policy.png">
      <img src="../../../assets/images/databricks/aws/databricks_secrets_manager_policy.png" alt="Configuring the access policy for the Secrets Manager">
    </a>
    <figcaption>Configuring the access policy for the Secrets Manager</figcaption>
  </figure>
</p>

### Azure

On Azure we currently do not support storing the API Key in a secret storage. Instead just store the API Key in a file in your Databricks workspace so you can access it when connecting to the Feature Store.

## Next Steps

Continue with the [configuration guide](configuration.md) to finalize the configuration of the Databricks Cluster to communicate with the Hopsworks Feature Store.
