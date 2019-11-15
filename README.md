# Policy to Restrict Vault AppRole Secret IDs
This repository includes a Sentinel policy that restricts the generation of secret IDs for Vault's AppRole method to requests that include metdata with an allowed hostname and the token_bound_cidrs parameter with an allowed CIDR.

It also includes test cases for use with the Sentinel Simulator.
