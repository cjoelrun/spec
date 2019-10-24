All exchanges in Tanker’s protocol are done in the form of blocks, representing the actions taken by users. Such actions can be, for example: the creation of a new device, the creation or update of a user group, or the sharing of an encryption key to a user.

Blocks contain a serialized payload, which contains different information depending on the nature of the block. Typical contents of a block payload are public keys, encrypted private or symmetric keys, and any data necessary to prove the block's validity. 

Every block is signed by its author, whose signature key must have been created in a previous block, creating a cryptographic signatures chain, similar to a blockchain’s structure.

## Block format

A block must be smaller than 4MiB (checked server-side only).

Blocks and payloads are serialized in a custom binary format.

- A variable buffer is a varint defining the length of the buffer followed by the buffer itself.
- A list is a varint defining the number of elements in it followed by the elements themselves.

| **Field name** | **Type**                | **Description**                                              |
| -------------- | ----------------------- | ------------------------------------------------------------ |
| version        | varint                  | The serialisation version                                    |
| index          | varint                  | The index of this block                                      |
| trustchain_id  | variable buffer         | The ID of the Trustchain this block belongs to               |
| nature         | varint                  | The nature of the block defining the type of the payload     |
| payload        | variable buffer         | The contents of the block, see payloads                      |
| author         | fixed buffer (32 bytes) | Hash of the block of the authority who has emitted this block. When the author block is a device, this is also the device id. |
| signature      | fixed buffer (64 bytes) | The signature of the block                                   |

## Block nature

The nature of a block indicates the type of action the block represents, and so the expected serialization format of the payload:

| **Nature**                          | **Nature value** | **Payload**                 |
| ----------------------------------- | ---------------- | --------------------------- |
| **trustchain_creation**             | 1                | TrustchainCreation          |
| **device_creation_v1**              | 2                | DeviceCreation v1           |
| **key_publish_to_device**           | 3                | KeyPublishToDevice          |
| **device_revocation_v1**            | 4                | DeviceRevocation            |
| **device_creation_v2**              | 6                | DeviceCreation v2           |
| **device_creation_v3**              | 7                | DeviceCreation v3           |
| **key_publish_to_user**             | 8                | KeyPublishToUser            |
| **device_revocation_v2**            | 9                | DeviceRevocation v2         |
| **user_group_creation_v1**          | 10               | UserGroupCreation v1        |
| **key_publish_to_user_group**       | 11               | KeyPublishToUserGroup v1    |
| **user_group_addition_v1**          | 12               | UserGroupAddition v1        |
| **key_publish_to_provisional_user** | 13               | KeyPublishToProvisionalUser |
| **provisional_identity_claim**      | 14               | ProvisionalIdentityClaim    |
| **user_group_creation_v2**          | 15               | UserGroupCreation v2        |
| **user_group_addition_v2**          | 16               | UserGroupAddition v2        |

## Payloads

### TrustchainCreation

| **Field name**       | **Type**                | **Description**                            |
| -------------------- | ----------------------- | ------------------------------------------ |
| public_signature_key | fixed buffer (32 bytes) | The public signature key of the Trustchain |

Notes: 

- The author field of the root block is always 0-filled.
- The appId is the hash of the root block, linking it to the public signature key

### DeviceCreation v1

| **Field name**                 | **Type**                | **Description**                                              |
| ------------------------------ | ----------------------- | ------------------------------------------------------------ |
| ephemeral_public_signature_key | fixed buffer (32 bytes) | The public key that can verify this block's signature        |
| user_id                        | fixed buffer (32 bytes) | The obfuscated user ID of the owner of this device           |
| delegation_signature           | fixed buffer (64 bytes) | The signature of the ephemeral public signature key and the user ID, by the block author |
| public_signature_key           | fixed buffer (32 bytes) | The public signature key of the added device                 |
| public_encryption_key          | fixed buffer (32 bytes) | The public encryption key of the added device                |

Possible author natures: 

- TrustchainCreation: for the first device of any user
- Device Creations: for subsequent devices

The device creation block's signatures:

- delegation_signature is the signature of the concatenation of the ephemeral_public_signature_key and the user_id, with the author key
- the block signature is the usual block signature, but signed with the ephemeral private key

### DeviceCreation v2

| **Field name**                 | **Type**                | **Description**                                              |
| ------------------------------ | ----------------------- | ------------------------------------------------------------ |
| last_reset                     | fixed buffer (32 bytes) | The hash of the last reset block of this user                |
| ephemeral_public_signature_key | fixed buffer (32 bytes) | The public key that can verify this block's signature        |
| user_id                        | fixed buffer (32 bytes) | The obfuscated user ID of the owner of this device           |
| delegation_signature           | fixed buffer (64 bytes) | The signature of the ephemeral public signature key and the user ID, by the block author |
| public_signature_key           | fixed buffer (32 bytes) | The public signature key of the added device                 |
| public_encryption_key          | fixed buffer (32 bytes) | The public encryption key of the added device                |

See DeviceCreation 1 for other details.

### DeviceCreation v3

| **Field name**                 | **Type**                | **Description**                                              |
| ------------------------------ | ----------------------- | ------------------------------------------------------------ |
| ephemeral_public_signature_key | fixed buffer (32 bytes) | The public key that can verify this block's signature        |
| user_id                        | fixed buffer (32 bytes) | The obfuscated user ID of the owner of this device           |
| delegation_signature           | fixed buffer (64 bytes) | The signature of the ephemeral public signature key and the user ID, by the block author |
| device_public_signature_key    | fixed buffer (32 bytes) | The public signature key of the added device                 |
| device_public_encryption_key   | fixed buffer (32 bytes) | The public encryption key of the added device                |
| user_key_pair                  | UserKeyPair             | The user public encryption key and the corresponding private key encrypted for the new device |
| is_ghost_device                | flag (last byte)        | This device is not a real device                             |

The last byte is a byte array with multiple flags. The byte array flags are in the same order as the fields in the DeviceCreation3 structure. is_ghost_device bitmask is 00000001

See DeviceCreation 1 for other details.

### DeviceRevocation v1

| **Field name** | **Type**                | **Description**       |
| -------------- | ----------------------- | --------------------- |
| device_id      | fixed buffer (32 bytes) | The revoked device ID |

Possible author natures: TrustchainCreation or Device Creations.

### DeviceRevocation v2

| **Field name** | **Type**                | **Description**                                              |
| -------------- | ----------------------- | ------------------------------------------------------------ |
| device_id      | fixed buffer (32 bytes) | The revoked device ID                                        |
| new_user_key   | NewUserKey              | The new user Public encryption Key and the corresponding private key encrypted for each of the user's devices |

Possible author natures: Device Creations

### UserGroupCreation v1

| **Field name**                                    | **Type**                 | **Description**                                              |
| ------------------------------------------------- | ------------------------ | ------------------------------------------------------------ |
| public_signature_key                              | fixed buffer (32 bytes)  | The signature key of the group                               |
| public_encryption_key                             | fixed buffer (32 bytes)  | The encryption key of the group                              |
| encrypted_group_private_signature_key             | fixed buffer (112 bytes) | The private signature key of the group encrypted for the group encryption key |
| encrypted_group_private_encryption_keys_for_users | list(GroupEncryptedKey)  | The new group keys encrypted for the users                   |
| self_signature                                    | fixed buffer (64 bytes)  | The signature of all non-signature fields, in that order with the group signature key |

Possible author natures: Device Creations.

### UserGroupCreation v2

| Field name                                                | Type                         | Description                                                  |
| --------------------------------------------------------- | ---------------------------- | ------------------------------------------------------------ |
| public_signature_key                                      | fixed buffer (32 bytes)      | The signature key of the group                               |
| public_encryption_key                                     | fixed buffer (32 bytes)      | The encryption key of the group                              |
| encrypted_group_private_signature_key                     | fixed buffer (112 bytes)     | The private signature key of the group encrypted for the group encryption key |
| encrypted_group_private_encryption_keys_for_users         | list(GroupEncryptedKey2)     | The new group keys encrypted for the users                   |
| pending_encrypted_group_private_encryption_keys_for_users | list(PendingGroupEncrypted2) | The new group keys encrypted for the pre-users               |
| self_signature                                            | fixed buffer (64 bytes)      | The signature of all non-signature fields, in that order with the group signature key |

### UserGroupAddition v1

| **Field name**                                    | **Type**                | **Description**                                           |
| ------------------------------------------------- | ----------------------- | --------------------------------------------------------- |
| group_id                                          | fixed buffer (32 bytes) | Group ID                                                  |
| previous_group_block                              | fixed buffer (32 bytes) | The hash of this group's last modification block          |
| encrypted_group_private_encryption_keys_for_users | list(GroupEncryptedKey) | The new group keys encrypted for the new users            |
| self_signature_with_current_key                   | fixed buffer (64 bytes) | The signature of all non-signature fields, in that order, |

Possible author natures: Device Creations.

This block can only add members, not remove them. The list of added members is the list of users in encrypted_group_keys_for_users. It does not rotate the group keys. Because it's so different from the UserGroupUpdate, we decided to make a different block.

### UserGroupAddition v2

| Field name                                            | Type                        | Description                                                  |
| ----------------------------------------------------- | --------------------------- | ------------------------------------------------------------ |
| group_id                                              | fixed buffer (32 bytes)     | Group ID                                                     |
| previous_group_block                                  | fixed buffer (32 bytes)     | The hash of this group's last modification block             |
| encrypted_group_private_encryption_keys_for_users     | list(GroupEncryptedKey2)    | The new group keys encrypted for the new users               |
| encrypted_group_private_encryption_keys_for_pre_users | list(GroupEncryptedPreKey2) | The new group keys encrypted for the new users               |
| self_signature_with_current_key                       | fixed buffer (64 bytes)     | The signature of all non-signature fields, in that order, with the current group signature key |

### KeyPublishToDevice

| **Field name** | **Type**                   | **Description**                                  |
| -------------- | -------------------------- | ------------------------------------------------ |
| recipient      | fixed buffer (32 bytes)    | The recipient deviceId                           |
| mac            | fixed buffer (16 bytes)    | The resource ID of the data this key can decrypt |
| key            | variable buffer (72 bytes) | The encrypted decryption key for the resource    |

The key is encrypted with a DH between the author's device's private key and the recipient's device's public key. Note that this field uses a variable buffer, but its effective size is fixed.

Possible author nature: Device Creations.

### KeyPublishToUser

| **Field name**       | **Type**                | **Description**                                  |
| -------------------- | ----------------------- | ------------------------------------------------ |
| recipient_public_key | fixed buffer (32 bytes) | The recipient user public key                    |
| resource_id          | fixed buffer (16 bytes) | The resource ID of the data this key can decrypt |
| encrypted_key        | EncryptedKey (80 bytes) | The encrypted decryption key for the resource    |

Possible author nature: Device Creations.

### KeyPublishToUserGroup 

| **Field name**             | **Type**                | **Description**                                  |
| -------------------------- | ----------------------- | ------------------------------------------------ |
| recipient_group_public_key | fixed buffer (32 bytes) | The recipient group public key                   |
| resource_id                | fixed buffer (16 bytes) | The resource ID of the data this key can decrypt |
| encrypted_key              | EncryptedKey (80 bytes) | The encrypted decryption key for the resource    |

Possible author nature: Device Creations.

### KeyPublishToProvisionalUser

| Field name                                            | Type                           | Description                                      |
| ----------------------------------------------------- | ------------------------------ | ------------------------------------------------ |
| recipient_app_public_preregistration_signature_key    | fixed buffer (32 bytes)        | The recipient preregistration public key         |
| recipient_tanker_public_preregistration_signature_key | fixed buffer (32 bytes)        | The recipient preregistration public key         |
| resource_id                                           | fixed buffer (16 bytes)        | The resource ID of the data this key can decrypt |
| encrypted_key                                         | DoubleEncryptedKey (128 bytes) | The encrypted decryption key for the resource    |

### ProvisionalIdentityClaim

| Field name                              | Type                     | Description                                            |
| --------------------------------------- | ------------------------ | ------------------------------------------------------ |
| user_id                                 | fixed buffer (32 bytes)  |                                                        |
| app_public_signature_key                | fixed buffer (32 bytes)  |                                                        |
| tanker_public_signature_key             | fixed buffer (32 bytes)  |                                                        |
| author_signature_by_app_key             | fixed buffer (64 bytes)  | The author device id signed by the preregistration key |
| author_signature_by_tanker_key          | fixed buffer (64 bytes)  | The author device id signed by the preregistration key |
| recipient_user_public_key               | fixed buffer (32 bytes)  |                                                        |
| encrypted_private_provisional_user_keys | fixed buffer (112 bytes) |                                                        |

## Substructures

### UserKeyPair

| **Field name**        | **Type**                | **Description**                                          |
| --------------------- | ----------------------- | -------------------------------------------------------- |
| public_encryption_key | fixed buffer (32 bytes) | The current public encryption key of the user            |
| encrypted_private_key | EncryptedKey (80 bytes) | The user private encryption key encrypted for the Device |

### NewUserKey

| **Field name**                      | **Type**                    | **Description**                                              |
| ----------------------------------- | --------------------------- | ------------------------------------------------------------ |
| public_encryption_key               | fixed buffer (32 bytes)     | The current public encryption key of the user                |
| previous_public_encryption_key      | fixed buffer (32 bytes)     | The previous public encryption key of the user or 0-array    |
| encrypted_key_for_previous_user_key | fixed buffer (80 bytes)     | The new user private key encrypted for previous user key or 0-array |
| encrypted_keys_for_devices          | list(EncryptedKeyForDevice) | The new user private key encrypted for other devices         |

### EncryptedKeyForDevice

| **Field name**        | **Type**                | **Description**                         |
| --------------------- | ----------------------- | --------------------------------------- |
| device_id             | fixed buffer (32 bytes) | The recipient device ID of this key     |
| encrypted_private_key | EncryptedKey (80 bytes) | The user key encrypted for the DeviceID |

### EncryptedKey

An EncryptedKey is a 80-byte buffer corresponding to a 32-bytes cleartext.

The buffer is encrypted as a sodium sealed box.

### DoubleEncryptedKey

An DoubleEncryptedKey is a 128-byte buffer corresponding to a 32-bytes cleartext, encrypted twice as a sodium sealed box.

### GroupEncryptedKey

| **Field name**                         | **Type**                | **Description**                                              |
| -------------------------------------- | ----------------------- | ------------------------------------------------------------ |
| public_user_encryption_key             | fixed buffer (32 bytes) | The public user key of the recipient of this key             |
| encrypted_group_private_encryption_key | fixed buffer (80 bytes) | The private encryption key of the group encrypted for the user key |

### GroupEncryptedKey2

| Field name                             | Type                    | Description                                                  |
| -------------------------------------- | ----------------------- | ------------------------------------------------------------ |
| user_id                                | fixed buffer (32 bytes) | The public user key of the recipient of this key             |
| public_user_encryption_key             | fixed buffer (32 bytes) | The public user key of the recipient of this key             |
| encrypted_group_private_encryption_key | fixed buffer (80 bytes) | The private encryption key of the group encrypted for the user key |

### PendingGroupEncryptedKey2

| Field name                             | Type                     | Description                                                  |
| -------------------------------------- | ------------------------ | ------------------------------------------------------------ |
| pending_app_public_signature_key       | fixed buffer (32 bytes)  | The public preshare key of the recipient of this key         |
| pending_tanker_public_signature_key    | fixed buffer (32 bytes)  | The public preshare key of the recipient of this key         |
| encrypted_group_private_encryption_key | fixed buffer (128 bytes) | The private encryption key of the group encrypted for the user key |

 