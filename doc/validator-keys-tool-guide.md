# Validator Keys Tool Guide

This guide explains how to set up a validator so its public key does not have to
change if the smcd config and/or server are compromised.

A validator uses a public/private key pair. The validator is identified by the
public key. The private key should be tightly controlled. It is used to:

*   sign tokens authorizing a smcd server to run as the validator identified
    by this public key.
*   sign revocations indicating that the private key has been compromised and
    the validator public key should no longer be trusted.

Each new token invalidates all previous tokens for the validator public key.
The current token needs to be present in the smcd config file.

Servers that trust the validator will adapt automatically when the token
changes.

## Validator Keys

When first setting up a validator, use the `validator-keys` tool to generate
its key pair:

```
  $ validator-keys create_keys
```

Sample output:
```
  Validator keys stored in /home/ubuntu/.smc/validator-keys.json
```

Keep the key file in a secure but recoverable location, such as an encrypted
USB flash drive. Do not modify its contents.

## Validator Token

After first creating the [validator keys](#validator-keys) or if the previous
token has been compromised, use the `validator-keys` tool to create a new
validator token:

```
  $ validator-keys create_token
```

Sample output:

```
  Update smcd.cfg file with these values:

  # validator public key: 59NJ7tuhPoEnfAff9svpF1es6zrbgXmVaE1YiJXhXj1DPL2C9pL3

  [validator_token]
  eyJtYW5pZmVzdCI6IkpBQUFBQUZ4SVFQdUlxUGR1V1FXU3gvOFRUa1l1blZBK0g4bUgyUUZp
  eml5U1RnMVZiZHdyWE1oQTJLWGpnQ1ZPTUNmNkdyUVFxSWttS0JicmNpOUZZMjVTR3FNcldl
  Y3BZYjdka2N3UlFJaEFMYzIrQjBNWlJ2QzhWcUR2ZkQ2YmF4bm83WlB0SDB3STc3WUVuaUN3
  VEttQWlBcUNIbURGYVZqQ1JuREZNbytTQ1JKOGhDODdTTVA1b2F0aDlteEpuZEJEbkFTUnpC
  RkFpRUFpNVJnS1RxdHlOalFaWitwL1hDeGF2a1RubStOblViMUJHUjFJWHlEVVYwQ0lCWGR3
  a0labVA5QUdWUENwN0htbkdoLzE1aVowU052K3ArUjh6Y082TzBvIiwidmFsaWRhdGlvbl9z
  ZWNyZXRfa2V5IjoiNTJFMzY3NzdGMUU2NDkyOEMzNTVGMTU0MjhDQTA3RTM2MjgxMzM4QzZF
  MUQ2OEIwNzU0RjRENjZCMUUyMTAyNCJ9
```

For a new validator, add the [validator_token] value to the smcd config file.
For a pre-existing validator, replace the old [validator_token] value with the
newly generated one. A valid config file may only contain one [validator_token]
value. After the config is updated, restart smcd.

There is a hard limit of 4,294,967,293 tokens that can be generated for a given
validator key pair.

## Key Revocation

If a validator private key is compromised, the key must be revoked permanently.
To revoke the validator key, use the `validator-keys` tool to generate a
revocation, which indicates to other servers that the key is no longer valid:

```
  $ validator-keys revoke_keys
```

Sample output:

```
  WARNING: This will revoke your validator keys!

  Update smcd.cfg file with these values and restart smcd:

  # validator public key: 59NJ7tuhPoEnfAff9svpF1es6zrbgXmVaE1YiJXhXj1DPL2C9pL3

  [validator_key_revocation]
  JP////9xIQPuIqPduWQWSx/8TTkYunVA+H8mH2QFiziySTg1VbdwrXASRzBFAiEAwAF/QTgp
  e/KpSoxIhIpmUat4WrvmncUd03Poshim4sYCIEqVWB5H3yfu1RHpjcsxPlc2VXyb0PLds+Gr
  kWEZ6Jd9
```

Add the `[validator_key_revocation]` value to this validator's config and
restart smcd. Rename the old key file and generate new [validator keys](#validator-keys) and
a corresponding [validator token](#validator-token).

## Signing data

The `validator-keys` tool can be used to sign arbitrary data with the validator
key.

```
  $ validator-keys sign_data "your data to sign"
```

Sample output:

```
  B91B73536235BBA028D344B81DBCBECF19C1E0034AC21FB51C2351A138C9871162F3193D7C41A49FB7AABBC32BC2B116B1D5701807BE462D8800B5AEA4F0550D
```

## Signing as publisher

The `validator-keys` tool can be used to sign a trusted validator list. In order to do that, you have to provide
a path to file containing an array of trusted validator manifests in JSON format.

```
  $ cat ~/unl_source.json
  [
     {
        "manifest" : "JAAAAAFxIQI319MYkVwlTXhi21KC3x4U6dKbxSvW1Wl4NYcg6990BnMhAqJKlj5cXOfP1wXxSfpDsghkcU2MB17k+/ocQNxc8LVIdkcwRQIhAIzjz7DTBLOWkOGE6enhqZlXSSBF4Q90V9bBch4Ok6R/AiBNuS+/UOi8eCVPZe6aP8gv4BCj7Gaz+VpIzTqQsnMeFXASRjBEAiAS45nGxTI4Yk9Y9PPD3PP72BUYQsxYePW6Qvh6bGdWSwIgA1wxW5uHaS4EnUdlvWg74Yvtk9KH8bWTYKQx1evQgoE=",
        "validation_public_key" : "0237D7D318915C254D7862DB5282DF1E14E9D29BC52BD6D56978358720EBDF7406"
     },
     {
        "manifest" : "JAAAAAFxIQPacgmcv+Lcqq4iXQwjtpw9k8KV9zG69ejZhtEhIwr4IHMhA/vHMnp7UcPDM+MmJhRn7ztyHhZLd00+xZGdA0x5c3J8dkcwRQIhAN/1fms0I7G5JKRaD5Bf9Pju0l1VQImXQxG0p+5wWA84AiAmS44K/f3TUmVToNy8Y/yTQA7LBRNUaXz8vnzJ7X2aMnASRzBFAiEAqn6skWpspnKW7b6q0rMCzdmEvxhg7SxCMtZJXjIar14CICPPEqm1ukaG3LnxRxv3CkG/nwbdVxavIxIzJBfTXkr1",
        "validation_public_key" : "03DA72099CBFE2DCAAAE225D0C23B69C3D93C295F731BAF5E8D986D121230AF820"
     },
     {
        "manifest" : "JAAAAAFxIQJHh/70IO54XbCMC9h1YhNvTc2xCgIE3bMu3wyLOleJoHMhA/VToKOZcERTrb062ce82p8gZC5QX+dtp5+XOEgwSkprdkYwRAIgbpL1zDYmx54yPyJ9xvdIHRoV6t6ixq65NNiSPAnOhrMCIH4V6jzJU4EUWjYlJuuy4v1P64PnK/grLiOcLhfWe3IdcBJGMEQCIHsDzRIEdZjzTJNXQU9rCJ4BvxIKpku6g8NmECZPcMUYAiAGulvbYOwQtO74GHbrPpeWEDzINXvSb+I+tfiVjgo5Mw==",
        "validation_public_key" : "024787FEF420EE785DB08C0BD87562136F4DCDB10A0204DDB32EDF0C8B3A5789A0"
     }
  ]
```

Signing itself is done using sign_list command:

```
  validator-keys sign_list ~/unl_source.json
```

Sample output:

```
  {"blob":"eyJleHBpcmF0aW9uIjo2MDgwODg1NjEsInNlcXVlbmNlIjoyLCJ2YWxpZGF0b3JzIjpbeyJtYW5pZmVzdCI6IkpBQUFBQUZ4SVFJMzE5TVlrVndsVFhoaTIxS0MzeDRVNmRLYnhTdlcxV2w0TlljZzY5OTBCbk1oQXFKS2xqNWNYT2ZQMXdYeFNmcERzZ2hrY1UyTUIxN2srL29jUU54YzhMVklka2N3UlFJaEFJemp6N0RUQkxPV2tPR0U2ZW5ocVpsWFNTQkY0UTkwVjliQmNoNE9rNlIvQWlCTnVTKy9VT2k4ZUNWUFplNmFQOGd2NEJDajdHYXorVnBJelRxUXNuTWVGWEFTUmpCRUFpQVM0NW5HeFRJNFlrOVk5UFBEM1BQNzJCVVlRc3hZZVBXNlF2aDZiR2RXU3dJZ0Exd3hXNXVIYVM0RW5VZGx2V2c3NFl2dGs5S0g4YldUWUtReDFldlFnb0U9IiwidmFsaWRhdGlvbl9wdWJsaWNfa2V5IjoiMDIzN0Q3RDMxODkxNUMyNTRENzg2MkRCNTI4MkRGMUUxNEU5RDI5QkM1MkJENkQ1Njk3ODM1ODcyMEVCREY3NDA2In0seyJtYW5pZmVzdCI6IkpBQUFBQUZ4SVFQYWNnbWN2K0xjcXE0aVhRd2p0cHc5azhLVjl6RzY5ZWpaaHRFaEl3cjRJSE1oQS92SE1ucDdVY1BETStNbUpoUm43enR5SGhaTGQwMCt4WkdkQTB4NWMzSjhka2N3UlFJaEFOLzFmbXMwSTdHNUpLUmFENUJmOVBqdTBsMVZRSW1YUXhHMHArNXdXQTg0QWlBbVM0NEsvZjNUVW1WVG9OeThZL3lUUUE3TEJSTlVhWHo4dm56SjdYMmFNbkFTUnpCRkFpRUFxbjZza1dwc3BuS1c3YjZxMHJNQ3pkbUV2eGhnN1N4Q010WkpYaklhcjE0Q0lDUFBFcW0xdWthRzNMbnhSeHYzQ2tHL253YmRWeGF2SXhJekpCZlRYa3IxIiwidmFsaWRhdGlvbl9wdWJsaWNfa2V5IjoiMDNEQTcyMDk5Q0JGRTJEQ0FBQUUyMjVEMEMyM0I2OUMzRDkzQzI5NUY3MzFCQUY1RThEOTg2RDEyMTIzMEFGODIwIn0seyJtYW5pZmVzdCI6IkpBQUFBQUZ4SVFKSGgvNzBJTzU0WGJDTUM5aDFZaE52VGMyeENnSUUzYk11M3d5TE9sZUpvSE1oQS9WVG9LT1pjRVJUcmIwNjJjZTgycDhnWkM1UVgrZHRwNStYT0Vnd1NrcHJka1l3UkFJZ2JwTDF6RFlteDU0eVB5Sjl4dmRJSFJvVjZ0Nml4cTY1Tk5pU1BBbk9ock1DSUg0VjZqekpVNEVVV2pZbEp1dXk0djFQNjRQbksvZ3JMaU9jTGhmV2UzSWRjQkpHTUVRQ0lIc0R6UklFZFpqelRKTlhRVTlyQ0o0QnZ4SUtwa3U2ZzhObUVDWlBjTVVZQWlBR3VsdmJZT3dRdE83NEdIYnJQcGVXRUR6SU5YdlNiK0krdGZpVmpnbzVNdz09IiwidmFsaWRhdGlvbl9wdWJsaWNfa2V5IjoiMDI0Nzg3RkVGNDIwRUU3ODVEQjA4QzBCRDg3NTYyMTM2RjREQ0RCMTBBMDIwNEREQjMyRURGMEM4QjNBNTc4OUEwIn1dfQ==","manifest":"JAAAAAJxIQPuIqPduWQWSx/8TTkYunVA+H8mH2QFiziySTg1VbdwrXMhA5s3Rv/ZluWGl1RzItV5ILi1MIY9tbG8Y8+mHH5vdaSHdkYwRAIgbE/AWMd6LvhSyXnfHlCIKPI9qjUgiU8vj5T+ey85hpQCIGqu02X20DOtlJql9cXhHxF1zt1cOtZm8QwSkzedD9+DcBJHMEUCIQDxGvGuosFipeuqCqBfnpaPME5Xj5FVugU+9SkaUq1PcAIgCR1ae6ziLzhzrLTrgB8dTwGMdSMIzj44ggHz+QqPeqc=","public_key":"03EE22A3DDB964164B1FFC4D3918BA7540F87F261F64058B38B249383555B770AD","signature":"304502210098E63CAC89FCE8A54718620E7176C7E20B52A5B40547114352EAFBA140ADCC56022007FFA5DACEA80692881D96FA60A8C64BD3F8737EA715513553324F8F3E55785B","version":1}
```

The resulting list may be then published in a reasonable way.
