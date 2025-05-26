# How to prepare an entry for the registry (NA policy script)

## Installing offchain token meta data tools doc

### 1. Install prerequisites

```
sudo apt update
sudo apt install -y wget tar
```

### 2. Download the latest v0.4.0.0 tarball

```
wget https://github.com/input-output-hk/offchain-metadata-tools/releases/download/v0.4.0.0/token-metadata-creator.tar.gz
wget https://github.com/input-output-hk/offchain-metadata-tools/releases/download/v0.4.0.0/metadata-validator-github.tar.gz

```

### 3. Extract

```
tar -xzf token-metadata-creator.tar.gz
tar -xzf metadata-validator-github.tar.gz
```

### 4. Install into your PATH

```
mkdir -p ~/.local/bin
mv token-metadata-creator metadata-validator-github ~/.local/bin/
```

```
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### 5. Verify

```
token-metadata-creator --help
metadata-validator-github –help
```

## Creating policy and policyId for the token

### 1. Make policy folder

```
mkdir policy
```

note
We don't navigate into this directory, and everything is done from our working directory.

First of all, we — again — need some key pairs:

### 2. Create key pair

```
cardano-cli address key-gen \
 --verification-key-file policy/policy.vkey \
 --signing-key-file policy/policy.skey
```

### 3. Creating policy.json

Use the echo command to create a policy.json (the first line uses > instead of >>, to create the file if not present and clear any existing contents):

```
echo "{" > policy/policy.json
echo " \"keyHash\": \"$(cardano-cli address key-hash --payment-verification-key-file policy/policy.vkey)\"," >> policy/policy.json
echo " \"type\": \"sig\"" >> policy/policy.json
echo "}" >> policy/policy.json
```

### 4. Generating Policy Id

```
cardano-cli conway transaction policyid --script-file ./policy/policy.script > policy/policyID
```

## Preparing an entry for the registry

### 1: Generate the 'subject'

PolicyId: "d4fece6b39f7cd78a3f036b2ae6508c13524b863922da80f68dd9ab7"
assetName: “USDM”

```
Base16 encode your assetName:
$ echo -n "USDM" | xxd -ps
```

5553444d

Concatenate the policyId with the base16-encoded assetName to obtain the 'subject' for your entry: d4fece6b39f7cd78a3f036b2ae6508c13524b863922da80f68dd9ab75553444d

### 2: Prepare a draft entry​

• Initialise a new draft entry for the subject using token metadata creator

```
token-metadata-creator entry --init d4fece6b39f7cd78a3f036b2ae6508c13524b863922da80f68dd9ab75553444d

```

This creates a draft JSON file named after your subject.

### 3: Add required fields

```
token-metadata-creator entry d4fece6b39f7cd78a3f036b2ae6508c13524b863922da80f68dd9ab75553444d \
 --name "USDM" \
 --description "A currency for the Plastiks." \
 --policy policy/policy.json
```

### 4: Add optional fields

```
token-metadata-creator entry d4fece6b39f7cd78a3f036b2ae6508c13524b863922da80f68dd9ab75553444d \
 --ticker "USDM" \
 --url "https://www.plastiks.io/" \
 --logo "usdm.png" \
 --decimals 6
```

### 5: Sign your metadata

Each metadata item must be signed with the key/s used to define your asset policy. For this example we assume only a single signing key is required to validate the monetary policy.json and that all metadata fields will be signed at once with the signing key file. Please refer to offchain-metadata-tools for more granular options.

```
token-metadata-creator entry d4fece6b39f7cd78a3f036b2ae6508c13524b863922da80f68dd9ab75553444d -a policy/policy.skey
```

### 6: Finalize your mapping

This will run some additional validations on your submission and check that it is considered valid.

```
token-metadata-creator entry d4fece6b39f7cd78a3f036b2ae6508c13524b863922da80f68dd9ab75553444d --finalize
```
