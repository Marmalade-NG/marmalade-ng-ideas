# Marmalade NG Collections Metadata

After discussing with Hypercent, it appears that Marmalade NG should be able to handle collection-wise metadata.
A marketplace should be able to display a cover image for a collection and other information.
  
This point is currently being discussed on Ethereum too: 
https://eips.ethereum.org/EIPS/eip-7572

I propose to add a metadata URI in the collection policy. Like NFT's metadata, this URI will be immutable.


## policy-collection proposal

I propose the following: 

##### Add an `uri` field in the collection schema.

```pact
  (defschema collection-sch
    id:string
    name:string
    size:integer
    max-size:integer
    creator:string
    creator-guard:guard
    uri:string
  )
```
##### Change the API of create collection: add a new parameter uri

This parameter is optional and can be left blank.
```pact
  (defun create-collection:bool (id:string name:string size:integer creator:string creator-guard:guard, uri:string)
```


##### Add a new function, allowing to set the uri for a specific collection.

Only if the previous uri was blank (ie: the uri has not been set-up anymore) Otherwise the URI keeps immutable.
Has to be signed by the collection creator.
This function in important because, we should allow an old collection created without a collection meta URI to add one.

```pact
(defun set-uri:bool (id:string new-uri:string)
```

##### Indeed this will break the backward compatubility of all functions that uses `collection-sch`.

##### As a consequence, policy-collection won't bless previous versions.  

##### Migration script
The database will be migrated with the governance rights. 
All existing collections will have a blank `uri` field added.


## Metadata format

Like NFT metadata, I strongly recommend to store them on an immutable media like ipfs or kdafs.

I propose to use the same format used by the ERC-7572 with the following differences:
- Remove collaborators. (in NG we have already the authenticated collection guard for that purpose)
- Add a properties field (similar to the one of NFTs metadata), for storing several various data.

Example:
```json
{
  "name": "Example Contract",
  "description": "Your description here",
  "image": "ipfs://QmTNgv3jx2HHfBjQX9RnKtxj2xv2xQCtbDXoRi5rJ3a46e",
  "banner_image": "ipfs://QmdChMVnMSq4U7oVKhud7wUSEZGnwuMuTY5rUQx57Ayp6H",
  "featured_image": "ipfs://QmS9m6e1E1NfioMM8dy1WMZNN2FRh2WDjeqJFWextqXCT8",
  "external_link": "https://project-website.com",
  "properties": {
    "authors": [
      { "name": "Jim Davis" }
    ],
    "discord_url": "https://discord.com/invite/my-poject",
    "telegram_url": "https://t.me/my-rpoject",
    "twitter_username": "ProjectTwitter"
  }
}
```
