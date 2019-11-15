# Policy to Restrict Vault AppRole Secret IDs
This repository includes a Sentinel policy that restricts the generation of secret IDs for Vault's AppRole method to requests that include metdata with an allowed hostname and the token_bound_cidrs parameter with an allowed CIDR.

Specifically, the `metadata` argument must include a key `hostname` with value `host1` and the `token_bound_cidrs` parameter must have the value `192.168.0.1/32`.

It also includes test cases for use with the Sentinel Simulator.

## Testing With the Sentinel Simulator
To test with the Sentinel Simulator, run one of these commands from the directory containing the policy restrict-approle-secret-id-generation.sentinel:
```
sentinel test
sentinel test -verbose
```

## Testing Against an Actual Vault Server
To test against an actual Vault server, please do the following:
1. Add the policy to the Vault server.  You can do this in the Vault UI by clicking the "Endpoint Governing Policies" link under the Policies menu and then clicking the "Create EGP Policy" button in the upper right corner. Enter "restrict-approle-secret-id-generation" as the name of the policy, paste the contents of restrict-approle-secret-id-generation.sentinel into the Policy field, set the enforcement level, enter `auth/approle/role/my-role/*` for the Paths, and click the "Create policy" button.
1. Enable the AppRole auth method with `vault auth enable approle`.
1. Create an AppRole role with `vault write auth/approle/role/my-role secret_id_bound_cidrs="192.168.0.0/16" token_bound_cidrs="192.168.0.0/16"`.
1. Create a Vault ACL policy as follows: You can do this in the Vault UI by clicking on the "ACL Policies" link under the Policies menu and then clicking the "Create ACL Policy" button. Enter a policy name like "generate-approle-secret-ids" and paste the policy snippet below into the Policy field. Then click the "Create policy" button.
1. Create a Vault token that is not a root token but does have the generate-approle-secret-ids policy with this command: `vault token create -policy=generate-approle-secret-ids`.
1. Export the Vault token returned by that token creation command with a command like `export VAULT_TOKEN=<token>`.

### Vault ACL Policy Snippet
The following Vault ACL policy snippet can be used in the Vault ACL policy "generate-approle-secret-ids" that you are instructed to create above.
```
path "auth/approle/role/my-role/secret-id" {
  capabilities = ["update"]
}
```

Finally, you can test the Sentinel policy with commands like the following.  All but the last should fail and print out specific messages indicating why they violated the Sentinel policy:

Provide empty metadata and leave out `token_bound_cidrs`:
```
vault write auth/approle/role/my-role/secret-id metadata='{}'
```
This fails with these messages:
```
token_bound_cidrs parameter is missing
hostname is missing from the metadata parameter or does not contain a valid host
```

Specify `host` in the metadata instead of `hostname` and leave out `token_bound_cidrs`:
```
vault write auth/approle/role/my-role/secret-id metadata='{"host": "host1"}'
```
This fails with these messages:
```
token_bound_cidrs parameter is missing
hostname is missing from the metadata parameter or does not contain a valid host
```

Specify `hostname` with value `host2` and leave out `token_bound_cidrs`:
```
vault write auth/approle/role/my-role/secret-id metadata='{"hostname": "host2"}'
```
This fails with these messages:
```
token_bound_cidrs parameter is missing
hostname is missing from the metadata parameter or does not contain a valid host
```

Specify `hostname` with value `host1` and leave out `token_bound_cidrs`:
```
vault write auth/approle/role/my-role/secret-id metadata='{"hostname": "host1"}'
```
This fails with the message:
```
token_bound_cidrs parameter is missing
```

Specify `hostname` with value `host1` and include `token_bounnd_cidrs` with value `192.168.0.2/32`:
```
vault write auth/approle/role/my-role/secret-id metadata='{"hostname": "host1"}' token_bound_cidrs="192.168.0.2/32"
```
This fails with the message:
```
token_bound_cidrs does not contain a valid CIDR
```

Specify `hostname` with value `host1` and include `token_bounnd_cidrs` with value `192.168.0.1/32`:
```
vault write auth/approle/role/my-role/secret-id metadata='{"hostname": "host1"}' token_bound_cidrs="192.168.0.1/32"
```
This succeeds and returns a secret ID similar to this:
```
Key                   Value
---                   -----
secret_id             a4425900-aa61-495e-12bd-c92429d2313b
secret_id_accessor    b8d0ceb0-774c-2a0f-d862-d76637f2ba54
```
