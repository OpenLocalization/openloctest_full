<properties 
	pageTitle="Get Started with Client-Side Encryption for Microsoft Azure Storage (Preview) | Microsoft Azure" 
	description="The Azure Storage Client Library for .NET preview offers support for client-side encryption and integration with Azure Key Vault. Client-side encryption offers maximum security for your Azure Storage applications, as your access keys are never available to the service. Client-side encryption is available for blobs, queues, and tables." 
	services="storage" 
	documentationCenter=".net" 
	authors="tamram" 
	manager="carolz" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.workload="storage" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/18/2015" 
	ms.author="tamram"/>


# Get Started with Client-Side Encryption for Microsoft Azure Storage (Preview)

## Overview

Welcome to the [preview of the new Azure Storage Client Library for .NET](https://www.nuget.org/packages/WindowsAzure.Storage/4.4.1-preview). This preview library contains new functionality to help developers encrypt data inside client applications before uploading to Azure Storage, and to decrypt data while downloading. The preview library also supports integration with Azure [Key Vault](http://azure.microsoft.com/services/key-vault/) for storage account key management.

## Encryption and decrpyption via the envelope technique

The processes of encryption and decryption follow the envelope technique.

### Encryption via the envelope technique

Encryption via the envelope technique works in the following way:

1. The Azure storage client library generates a content encryption key (CEK), which is a one-time-use symmetric key.
2. User data is encrypted using this CEK.
3. The CEK is then wrapped (encrypted) using the key encryption key (KEK). The KEK is identified by a key identifier and can be an asymmetric key pair or a symmetric key and can be managed locally or stored in Azure Key Vaults. 
	
	The storage client library itself never has access to KEK. The library invokes the key wrapping algorithm that is provided by Key Vault. Users can choose to use custom providers for key wrapping/unwrapping if desired.

4. The encrypted data is then uploaded to the Azure Storage service. The wrapped key along with some additional encryption metadata is either stored as metadata (on a blob) or interpolated with the encrypted data (queue messages and table entities).

### Decryption via the envelope technique

Decryption via the envelope technique works in the following way:

1. The client library assumes that the user is managing the key encryption key (KEK) either locally or in Azure Key Vaults. The user does not need to know the specific key that was used for encryption. Instead, a key resolver which resolves different key identifiers to keys can be set up and used.
2. The client library downloads the encrypted data along with any encryption material that is stored on the service.
3. The wrapped content encryption key (CEK) is then unwrapped (decrypted) using the key encryption key (KEK). Here again, the client library does not have access to KEK. It simply invokes the custom or Key Vault provider’s unwrapping algorithm.
4. The content encryption key (CEK) is then used to decrypt the encrypted user data.

## Encryption Mechanism

The storage client library uses [AES](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard) in order to encrypt user data. Specifically, [Cipher Block Chaining (CBC)](http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher-block_chaining_.28CBC.29) mode with AES. Each service works somewhat differently, so we will discuss each of them here.

### Blobs

In the preview version, the client library supports encryption of whole blobs only. Specifically, encryption is supported when users use **UploadFrom*** methods or **BlobWriteStream**. For downloads, both full and range downloads are supported.

During encryption, the client library will generate a random Initialization Vector (IV) of 16 bytes, together with a random content encryption key (CEK) of 32 bytes, and perform envelope encryption of the blob data using this information. The wrapped CEK and some additional encryption metadata are then stored as blob metadata along with the encrypted blob on the service. 

> [AZURE.WARNING] If you are editing or uploading your own metadata for the blob, you need to ensure that this metadata is preserved. If you upload new metadata without this metadata, the wrapped CEK, IV and other metadata will be lost and the blob content will never be retrievable again.

Downloading an encrypted blob involves retrieving the content of the entire blob using the **DownloadTo***/**BlobReadStream** convenience methods. The wrapped CEK is unwrapped and used together with the IV (stored as blob metadata in this case) to return the decrypted data to the users.

Downloading an arbitrary range (**DownloadRange*** methods) in the encrypted blob involves adjusting the range provided by users in order to get a small amount of additional data that can be used to successfully decrypt the requested range.

All blob types (block blobs and page blobs) can be encrypted/decrypted using this scheme.

### Queues

Since queue messages can be of any format, the client library defines a custom format that includes the Initialization Vector (IV) and the encrypted content encryption key (CEK) in the message text.

During encryption, the client library generates a random IV of 16 bytes along with a random CEK of 32 bytes and performs envelope encryption of the queue message text using this information. The wrapped CEK and some additional encryption metadata are then added to the encrypted queue message. This modified message (shown below) is stored on the service.

	<MessageText>{"EncryptedMessageContents":"6kOu8Rq1C3+M1QO4alKLmWthWXSmHV3mEfxBAgP9QGTU++MKn2uPq3t2UjF1DO6w","EncryptionData":{…}}</MessageText>

During decryption, the wrapped key is extracted from the queue message and unwrapped. The IV is also extracted from the queue message and used along with the unwrapped key to decrypt the queue message data. Note that the encryption metadata is small (under 500 bytes), so while it does count toward the 64KB limit for a queue message, the impact should be manageable.

### Tables

In the preview version, the client library supports encryption of entity properties for insert and replace operations. 

>[AZURE.NOTE] Merge is not currently supported. Since a subset of properties may have been encrypted previously using a different key, simply merging the new properties and updating the metadata will result in data loss. Merging either requires making extra service calls to read the pre-existing entity from the service, or using a new key per property, both of which are not suitable for performance reasons.

Table data encryption works as follows:  

1. Users specify the properties that should be encrypted.
2. The client library generates a random Initialization Vector (IV) of 16 bytes along with a random content encryption key (CEK) of 32 bytes for every entity, and perform envelope encryption on the individual properties that should be encrypted by deriving a new IV per property.
3. The wrapped CEK and some additional encryption metadata are then stored as two additional reserved properties. The first reserved property (_ClientEncryptionMetadata1) is a string property that holds the information about IV, version, and wrapped key. The other reserved property (_ClientEncryptionMetadata2) is a binary property that holds the information about the properties that are encrypted.
4. Due to these additional reserved properties required for encryption, users may now have only 250 custom properties instead of 252. The total size of the entity must be less than 1MB.

Note that only string properties can be encrypted. If other types of properties are to be encrypted, they must be converted to strings.

For tables, in addition to the encryption policy, users must specify the properties that should be encrypted. This can be done by either specifying an [EncryptProperty] attribute (for POCO entities that derive from TableEntity) or an encryption resolver in request options. An encryption resolver is a delegate that takes a partition key, row key, and property name and returns a Boolean that indicates whether that property should be encrypted. During encryption, the client library will use this information to decide whether a property should be encrypted while writing to the wire. The delegate also provides for the possibility of logic around how properties are encrypted. (For example, if X, then encrypt property A; otherwise encrypt properties A and B.) Note that it is not necessary to provide this information while reading or querying entities.

### Batch Operations

In batch operations, the same KEK will be used across all the rows in that batch operation because the client library only allows one options object (and hence one policy/KEK) per batch operation. However, the client library will internally generate a new random IV and random CEK per row in the batch. Users can also choose to encrypt different properties for every operation in the batch by defining this behavior in the encryption resolver.

### Queries

To perform query operations, you must specify a key resolver that is able to resolve all the keys in the result set. If an entity contained in the query result cannot be resolved to a provider, the client library will throw an error. For any query that performs server side projections, the client library will add the special encryption metadata properties (_ClientEncryptionMetadata1 and _ClientEncryptionMetadata2) by default to the selected columns.

## Azure Key Vault

Azure Key Vault (Preview) helps safeguard cryptographic keys and secrets used by cloud applications and services. By using Azure Key Vault, users can encrypt keys and secrets (such as authentication keys, storage account keys, data encryption keys, .PFX files, and passwords) by using keys that are protected by hardware security modules (HSMs). For more information, see [What is Azure Key Vault?](../articles/key-vault-whatis.md).

The storage client library uses the Key Vault core library in order to provide a common framework across Azure for managing keys. Users also get the additional benefit of using the Key Vault extensions library. The extensions library provides useful functionality around simple and seamless Symmetric/RSA local and cloud key providers as well as with aggregation and caching.

### Interface and dependencies

There are three Key Vault packages:

- Microsoft.Azure.KeyVault.Core contains the IKey and IKeyResolver. It is a small package with no dependencies. The storage client librarys for .NET and Windows Phone define it as a dependency.
- Microsoft.Azure.KeyVault contains the Key Vault REST client.
- Microsoft.Azure.KeyVault.Extensions contains extension code that includes implementations of cryptographic algorithms and an RSAKey and a SymmetricKey. It depends on the Core and KeyVault namespaces and provides functionality to define an aggregate resolver (when users want to use multiple key providers) and a caching key resolver. Although the storage client library does not directly depend on this package, if users wish to use Azure Key Vault to store their keys or to use the Key Vault extensions to consume the local and cloud cryptographic providers, they will need this package.

Key Vault is designed for high-value master keys, and throttling limits per Key Vault are designed with this in mind. When performing client-side encryption with Key Vault, the preferred model is to use symmetric master keys stored as secrets in Key Vault and cached locally. Users must do the following:

1. Create a secret offline and upload it to Key Vault.
2. Use the secret's base identifier as a parameter to resolve the current version of the secret for encryption and cache this information locally. Use CachingKeyResolver for caching; users are not expected to implement their own caching logic.
3. Use the caching resolver as an input while creating the encryption policy.

More information regarding Key Vault usage can be found in the [encryption code samples](https://github.com/Azure/azure-storage-net/tree/preview/Samples/GettingStarted/EncryptionSamples).

### Best practices

Encryption support is available only in the storage client libraries for .NET and Windows Phone. Windows Runtime does not currently support encryption. Additionally, Key Vault extensions are not currently supported for Windows Phone. If you want to use storage client encryption on phone, you will need to implement your own key providers. Also, due to a limitation in the Windows Phone .NET platform, page blob encryption is currently not supported on Windows Phone.

>[AZURE.IMPORTANT] Be aware of these important points when using the preview library:
>
>- Do not use the preview library for production data. In the future, changes to the library will affect the schemas used. Decryption of data that has been encrypted with the preview library is not guaranteed in future versions.  
>- When reading from or writing to an encrypted blob, use full blob upload commands and range/full blob download commands. Avoid writing to an encrypted blob using protocol operations such as Put Block, Put Block List, Write Pages, or Clear Pages; otherwise you may corrupt the encrypted blob and make it unreadable.
>- For tables, a similar constraint exists. Be careful to not update encrypted properties without updating the encryption metadata.
>- If you set metadata on the encrypted blob, you may overwrite the encryption-related metadata required for decryption, since seting metadata is not additive. This is also true for snapshots; avoid specifying metadata while creating a snapshot of an encrypted blob.


## Client API / Interface

While creating an EncryptionPolicy object, users can provide only a Key (implementing IKey), only a resolver (implementing IKeyResolver), or both. IKey is the basic key type that is identified using a key identifier and that provides the logic for wrapping/unwrapping. IKeyResolver is used to resolve a key during the decryption process. It defines a ResolveKey method that returns an IKey given a key identifier. This provides users the ability to choose between multiple keys that are managed in multiple locations.

- For encryption, the key is used always and the absence of a key will result in an error.
- For decryption:
	- The key resolver is invoked if specified to get the key. If the resolver is specified but does not have a mapping for the key identifier, an error is thrown.
	- If resolver is not specified but a key is specified, the key identifier on the key is against what is stored for the service.

The [encryption samples](https://github.com/Azure/azure-storage-net/tree/preview/Samples/GettingStarted/EncryptionSamples) demonstrate a more detailed end-to-end scenario for blobs, queues and tables, along with Key Vault integration.

### Blobs

Users will create a **BlobEncryptionPolicy** object and set it in the request options (per API or at a client level by using **DefaultRequestOptions**). Everything else will be handled by the client library internally.

	// Create the IKey used for encryption.
 	RsaKey key = new RsaKey("private:key1" /* key identifier */);
  
 	// Create the encryption policy to be used for upload and download.
 	BlobEncryptionPolicy policy = new BlobEncryptionPolicy(key, null);
  
 	// Set the encryption policy on the request options.
 	BlobRequestOptions options = new BlobRequestOptions() { EncryptionPolicy = policy };
  
 	// Upload the encrypted contents to the blob.
 	blob.UploadFromStream(stream, size, null, options, null);
  
 	// Download and decrypt the encrypted contents from the blob.
 	MemoryStream outputStream = new MemoryStream();
 	blob.DownloadToStream(outputStream, null, options, null);

### Queues

Users will create a **QueueEncryptionPolicy** object and set it in the request options (per API or at a client level by using **DefaultRequestOptions**). Everything else will be handled by the client library internally.


	// Create the IKey used for encryption.
 	RsaKey key = new RsaKey("private:key1" /* key identifier */);
  
 	// Create the encryption policy to be used for upload and download.
 	QueueEncryptionPolicy policy = new QueueEncryptionPolicy(key, null);
  
 	// Add message
 	QueueRequestOptions options = new QueueRequestOptions() { EncryptionPolicy = policy };
 	queue.AddMessage(message, null, null, options, null);
  
 	// Retrieve message
 	CloudQueueMessage retrMessage = queue.GetMessage(null, options, null);

### Tables

In addition to creating an encryption policy and setting it on request options, users will have to specify an **EncryptionResolver** in **TableRequestOptions** or set attributes on the entity.

#### Using the resolver


	// Create the IKey used for encryption.
 	RsaKey key = new RsaKey("private:key1" /* key identifier */);
  
 	// Create the encryption policy to be used for upload and download.
 	TableEncryptionPolicy policy = new TableEncryptionPolicy(key, null);
  
 	TableRequestOptions options = new TableRequestOptions() 
 	{ 
    	EncryptionResolver = (pk, rk, propName) =>
     	{
        	if (propName == "foo")
         	{
            	return true;
         	}
         	return false;
     	},
     	EncryptionPolicy = policy
 	};
  
 	// Insert Entity
 	currentTable.Execute(TableOperation.Insert(ent), options, null);
  
 	// Retrieve Entity
 	// No need to specify an encryption resolver for retrieve
 	TableRequestOptions retrieveOptions = new TableRequestOptions() 
 	{
    	EncryptionPolicy = policy
 	};
  
 	TableOperation operation = TableOperation.Retrieve(ent.PartitionKey, ent.RowKey);
 	TableResult result = currentTable.Execute(operation, retrieveOptions, null);

#### Using attributes

As mentioned above, if the entity implements TableEntity, then the properties can be decorated with the [EncryptProperty] attribute instead of specifying the EncryptionResolver.

	[EncryptProperty]
 	public string EncryptedProperty1 { get; set; }

## Next steps

[Client-Side Encryption for Microsoft Azure Storage – Preview](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/04/28/client-side-encryption-for-microsoft-azure-storage-preview.aspx)  
Download the [Azure Storage Client Library for .NET NuGet package](http://www.nuget.org/packages/WindowsAzure.Storage/4.4.0-preview)   
Download the [Azure Storage Client Library for .NET Source Code](https://github.com/Azure/azure-storage-net/tree/preview) from GitHub  
Download the Azure Key Vault NuGet [Core](http://www.nuget.org/packages/Microsoft.Azure.KeyVault.Core/), [Client](http://www.nuget.org/packages/Microsoft.Azure.KeyVault/), and [Extensions](http://www.nuget.org/packages/Microsoft.Azure.KeyVault.Extensions/) packages  
Visit the [Azure Key Vault Documentation](../articles/key-vault-whatis.md) 
