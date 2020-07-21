# tyronZIL DID-create operation

A tyronZIL DID-create operation is conformant with a Sidetree Create operation, and its goal is to generate a brand new tyronZIL Decentralized Identifier.

Follow these steps:

## 1. Verification methods

tyronZIL supports 3 verification methods: 'publicKey', 'operation' and 'recovery'.

1.1 Under the property 'publicKey', assing to its value an array of keys of type [PublicKeyModel](../../implementation/models.md#public-key-model), generated using the [operation key pair](../../sidetree.md#operation-key-pair):

```publicKeys: PublicKeyModel[]```

1.2 Under the property 'operation', assign to its value an [Operation](../../implementation/models.md#sidetree-verification-methods) verification method object:

```operation: Operation```

1.3 Under the property 'recovery', assign to its value a [Recovery](../../implementation/models.md#sidetree-verification-methods) verification method object:

```recovery: Recovery```

## 2. Public key commitments

2.1 For the [public key commitments](../../sidetree.md#public-key-commitment), first, generate two key-pairs:

```[UPDATE_KEY, UPDATE_PRIVATE_KEY]```

```[RECOVERY_KEY, RECOVERY_PRIVATE_KEY]```

2.2 Use the ```UPDATE_KEY``` to generate the update-commitment:

```UPDATE_COMMITMENT``` = [Multihash](../../sidetree.md#hash-protocol).canonicalize[ThenHash](../../sidetree.md#commitment-hash)[ThenEncode](../../sidetree.md#data-encoding-scheme)(```UPDATE_KEY```)

2.2 Use the ```RECOVERY_KEY``` to generate the recovery-commitment:

```RECOVERY_COMMITMENT``` = [Multihash](../../sidetree.md#hash-protocol).canonicalize[ThenHash](../../sidetree.md#commitment-hash)[ThenEncode](../../sidetree.md#data-encoding-scheme)(```RECOVERY_KEY```)

## 3. Service endpoints

Under the property 'service', assign to its value an array of service endpoint objects of type [ServiceEndpointModel](../../implementation/models.md#service-endpoint-model):

```service: ServiceEndpointModel[]```

## 4. Document model

Generate a document of type [DocumentModel](../../implementation/models.md#document-model):

```js
DOCUMENT = {  
    public_keys: publicKeys,  
    service_endpoints: service  
}
```

## 5. DID state patch

Put the previously generated document inside of a [DID state patch](../../sidetree.md#did-state-patch). For the DID-create operation, the patch action is 'replace':

```js
PATCH = {  
    action: PatchAction.Replace,  
    document: DOCUMENT  
}
```

> ```PATCH``` is of type [PatchModel](../../implementation/models.md#patch-model), with a Replace [PatchAction](../../implementation/models.md#patch-action)

## 6. Create Operation Delta Object

6.1 Using the DID state patch and the update-commitment, generate an instance of the Create Operation Delta Object as follows:

```js
DELTA = {
    patches: [PATCH],  
    updateCommitment: UPDATE_COMMITMENT  
}
```

> ```DELTA``` is of type [DeltaModel](../../implementation/models.md#delta-model)

Then apply the following operations to the object:

6.2 Stringify it and turn it into a buffer  
6.3 Encode it with the [data encoding scheme](../../sidetree.md#data-encoding-scheme) as ```ENCODED_DELTA```  
6.4 Hash it with the [hash algorithm](../../sidetree.md#hash-algorithm) & [hash protocol](../../sidetree.md#hash-protocol) and then encode it again as ```DELTA_HASH```

## 7. Create Operation Suffix Data Object

7.1 Using the ```DELTA_HASH``` from the previous step, and the recovery-commitment, generate an instance of the Create Operation Suffix Data Object as follows:

```js
SUFFIX_DATA = {  
    delta_hash: DELTA_HASH,  
    recovery_commitment: RECOVERY_COMMITMENT  
}
```

> ```SUFFIX_DATA``` is of type [SuffixDataModel](../../implementation/models.md#suffix-data-model)

Then stringify it and encode it with the [data encoding scheme](../../sidetree.md#data-encoding-scheme) as ```ENCODED_SUFFIX_DATA```.

## 8. Sidetree request

8.1 Generate the following object:

```js
SIDETREE_REQUEST = {  
    suffix_data: ENCODED_SUFFIX_DATA,  
    type: OperationType.Create,  
    delta: ENCODED_DELTA  
}
```

8.2 Stringify it and turn it into a buffer as ```OPERATION_BUFFER```  
8.3 Send the ```OPERATION_BUFFER``` to the Sidetree's library CreateOperation, which returns a class with the following properties:

- The original request buffer sent by the requester:

```operationBuffer: Buffer```

- The unique suffix of the DID, globally unique:

```didUniqueSuffix: string```

- The type of operation:

```type: OperationType.Create```

- The data used to generate the unique DID suffix:

```suffixData: SuffixDataModel```

- The encoded string of the suffix data:

```encodedSuffixData: string```

- The Create Operation Delta Object:

```delta: DeltaModel | undefined```

- The encoded string of the delta:

```encodedDelta: string | undefined```

## 9. tyronZIL DID-create operation result

The return value of a tyronZIL-js DID-create operation is an instance of the class [DidCreate](https://github.com/julio-cabdu/tyronZIL-js/tree/master/src/lib/did-operations/did-create.ts), which includes all the previously mentioned properties plus additional such as public, private keys, commitments and service endpoints.