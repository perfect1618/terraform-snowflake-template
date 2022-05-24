# Terraforming Snowflake

## steps

#### 1. Create a repository for Terraforming Snowflake

#### 2. Create a Service User for Terraform

We will create a user account separate from your own that uses key-pair authentication. This is required due to the provider's limitations around caching credentials and the lack of support for 2FA. Service accounts and key pair are how most CI/CD pipelines run Terraform.

##### 2.a Create an RSA key for Authentication

It creates the private and public keys we use to authenticate the service account for Terraform.

```
$ cd ~/.ssh
$ openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out snowflake_tf_snow_key.p8 -nocrypt
$ openssl rsa -in snowflake_tf_snow_key.p8 -pubout -out snowflake_tf_snow_key.pub
```

Copy the content of `snowflake_tf_snow_key.pub` file and save it into a place, we will use it in the next step.

##### 2.b Create the User for Terraform in Snowflake

Log in to the Snowflake console to create user for terraform: paste over the content of `snowflake_tf_snow_key.pub` to the value of `RSA_PUBLIC_KEY` in the SQL, create the user account with the following SQL using your `ACCOUNTADMIN` role.

```
CREATE USER tfsnow RSA_PUBLIC_KEY=${PASTE_RSA_PUBLIC_KEY_HERE} DEFAULT_ROLE=PUBLIC MUST_CHANGE_PASSWORD=FALSE;

GRANT ROLE SYSADMIN TO USER tfsnow;
GRANT ROLE SECURITYADMIN TO USER tfsnow;

DESCRIBE USER tfsnow;
```

An important security best practice is to limit all user accounts to least-privilege access. In a production environment, this key should also be secured with a secrets management solution like Hashicorp Vault, Azure Key Vault, or AWS Secrets Manager.

#### 3. Setup Terraform Authentication

We need to pass provider information via environment variables and input variables so that Terraform can authenticate as the user.

##### 3.a Add Account Information to Environment

Find your account locator and region from the web URL of snowflake.

e.g. `https://app.snowflake.com/us-central1.gcp/xd12345` can get the region as `us-central1.gcp` and the account identifier as `xd12345`

If you plan on working on this or other projects in multiple shells, it may be convenient to put these env vars in a `snow.env` file that you can source or put it in your `.bashrc` or `.zshrc` file. We expect you to run future Terraform commands in a shell with those set.

```
$ export SNOWFLAKE_USER=tfsnow
$ export SNOWFLAKE_PRIVATE_KEY_PATH="~/.ssh/snowflake_tf_snow_key.p8"
$ export SNOWFLAKE_ACCOUNT=xd12345
$ export SNOWFLAKE_REGION=us-central1.gcp
```
