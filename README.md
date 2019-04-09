# SimpleAssets  

A simple standard for digital assets (both fungible and NFTs - non-fungible tokens) for EOSIO blockchains   
by [CryptoLions](https://CryptoLions.io)  
  
web: http://simpleassets.io  
Git: https://github.com/CryptoLions/SimpleAssets  

Presentation:  https://medium.com/@cryptolions/introducing-simple-assets-b4e17caafaa4

Events Receiver Example for authors: https://github.com/CryptoLions/SimpleAssets-EventReceiverExample  

**WARNING!!! CDT currently has a bug that doesn't allow compilation on v1.6.1.
  1.5.0 also has a bug "Segmentation fault (core dumped)", but only with abi generation.
  Recommendation: Use 1.5.0 for contract compilation and use our abi**
  
---------------------------  

There are two ways to use Simple Assets:  
  
1) As an "ownership authority".  When deployed on an EOSIO chain, Simple Assets will be a separate contract which other Dapps can call to manage their digital assets.  This serves as an additional guarantee to users of the Dapp that the ownership of assets is managed by a reputable outside authority, and that once created, the Dapp can only manage the asset's mdata.  All the ownership-related functionality exists outside the game.  
  
   We are in the process of creating a DAC which will curate updates to Simple Assets after deployment to the EOSIO mainnet.  
  
2) Dapps can Deploy their own copy of Simple Assets and make modifications to have greater control of functionality.  We consider this an example of a dapp being its own "ownership authority."  Before deploying, Simple Assets should be modified to prevent anyone from making assets.  
---------------------------
## Scope:
1. [ChangeLog](#change-log-v032)
2. [Contract actions](#contract-actions)
3. [Data Structures](#data-structures)
4. [EXAMPLES: how to use Simple Assets in smart contracts](#examples-how-to-use-simple-assets-in-smart-contracts)

---------------------------  
## Change Log v0.4.0

**Easily find fungible token information (fungible tokens have scope author):**
- new field `author` in `account` table for FT. (makes it easier to find fungible token information)

**More fungible token information:**
- new field `data` in `currency_stats` table - stringify json which might include keys `img`, `name` (recommended for better displaying by markets)
- new parameter `data` in `createf` action
- new action `updatef` to change FT `data`

**Offer/claim fungible tokens**
- new table `sofferf` to use for `offer`/`calim` FT
- new actions `offerf`, `cancelofferf` and `claimf`
- on `closef` check if no open offers (internal)

**Containerizing assets**  
**WARNING!!! CDT currently has a bug that doesn't allow compilation (v1.6.1).  
  1.5.0 also has a bug "Segmentation fault (core dumped)", but only with abi generation.
  Recommendation: Use 1.5.0 for contract compilation and use our abi**
- new fields `container` and `containerf` in nft asset structure for attaching and detaching other NFT or FT
- new actions `attach`, `detach`
- new actions `attachf`, `detachf`

**misc**
- fields renamed `lastid` -> `lnftid`, `spare`->`defid` (internal usage) in table `global` 
- field `offeredTo` renamed to `offeredto` in table `soffer`


## Change Log v0.3.2
- Added `memo` parameter to action `offer`;  
- Added `memo` parameter to action `delegate`;


## Change Log v0.3.1
- Internal action for NFT `createlog` added. Used by create action to log assetid so that third party explorers can easily get new asset ids and other information.
- New singelton table `tokenconfigs` added. It helps external contracts parse actions and tables correctly (Usefull for decentralized exchanges, marketplaces and other contracts that use multiple tokens).
  Marketplaces, exchanges and other reliant contracts will be able to view this info using the following code.
	```
	Configs configs("simpl5assets"_n, "simpl5assets"_n.value);
	configs.get("simpl5assets"_n);
	```
- added action `updatever`. It updates version of this SimpleAstes deployment for 3rd party wallets, marketplaces, etc;
- New examples for Event notifications: https://github.com/CryptoLions/SimpleAssets-EventReceiverExample  


## Change Log v0.3.0
- Added event notifications using deferred transaction. Assets author will receive notification on assets create, transfer, claim or burn. To receive it please add next action to your author contract:  
	```
        ACTION saecreate   ( name owner, uint64_t assetid );  
        ACTION saetransfer ( name from, name to, std::vector<uint64_t>& assetids, std::string memo );  
        ACTION saeclaim    ( name account, std::vector<uint64_t>& assetids );  
        ACTION saeburn     ( name account, std::vector<uint64_t>& assetids, std::string memo );  
	```
- `untildate` parameter changed to `period` (in seconds) for actions `delegate` and table `sdelegates`  


## Change Log v0.2.0
### Added Fungible Token tables and logic using eosio.token contract but with some changes
- New actions and logic: `createf`, `issuef`, `transferf`, `burnf`, `openf`, `closef`
- added new tables `stat(supply, max_supply, issuer, id)` and `accounts (id, balance)`. 
- scope for stats table (info about fungible tokens) changed to author
- primary index for `accounts` table is uniq id created on createf action and stored in stats table.
- added  `createf` action for fungible token with parametr `authorctrl` to `stats` table. If true(1) allows token author (and not just owner) to burnf and transferf. Cannot be changed after creation!
- Ricardian contracts updated
- more usage examples below


## Change Log v0.1.1
Misc
- sdelagate table structure renamed to sdelegate (typo)
- create action parameters renamed: requireClaim -> requireclaim
- assetID action parameter renamed in all actions to assetid

Borrowing Assets
- sdelegate table - added new field: untildate
- delegate action added parameters untildate.  Action does a simple check if parameter was entered correctly (either zero or in the future).
- undelegate will not work until untildate (this guarantees a minimum term of the asset loan).
- allow transfer asset back (return) if its delegated, sooner than untiltime  (borrower has option ton return early)

Batch Processing
- claim action: assetid parameter changed to array of assetsids. Multiple claim logic added.
- offer action: assetid parameter changed to array of assetsids. Multiple offer logic added.
- canceloffer action: assetid parameter changed to array of assetsids. Multiple cancelation logic added.
- transfer action: assetid parameter changed to array of assetsids. Multiple assets transfer logic added.
- burn action: assetid parameter changed to array of assetsids. Multiple burning logic added.
- delegate/undelegate action: assetid parameter changed to array of assetsids. Multiple delegation/undelegation logic added.


------------------------  

# Contract actions  
A description of each parameter can be found here:  
https://github.com/CryptoLions/SimpleAssets/blob/master/include/SimpleAssets.hpp  

```
 # -- For Non-Fungible Tokens ---
 
 create			(author, category, owner, idata, mdata, requireсlaim)  
 update			(author, owner, assetid, mdata)  
 transfer		(from, to , [assetid1,..,assetidn], memo)  
 burn			(owner, [assetid1,..,assetidn], memo)  
 
 offer			(owner, newowner, [assetid1,..,assetidn], memo)  
 canceloffer		(owner, [assetid1,..,assetidn])  
 claim			(claimer, [assetid1,..,assetidn])  
  
 regauthor		(name author, data, stemplate)  
 authorupdate		(author, data, stemplate)  
 
 delegate		(owner, to, [assetid1,..,assetidn], period, memo)  
 undelegate		(owner, from, [assetid1,..,assetidn])  
 
 attach			(owner, assetidc, [assetid1,..,assetidn])
 detach			(owner, assetidc, [assetid1,..,assetidn])
 
 attachf		(owner, author, quantity, assetidc)
 detachf		(owner, author, quantity, assetidc)
 
 # -- For Fungible Tokens ---
 
 createf		(author, maximum_supply, authorctrl, data)
 updatef		(author, sym, data)
 issuef			(to, author, quantity, memo)
 transferf		(from, to, author, quantity, memo)
 burnf			(from, author, quantity, memo)

 offerf			(owner, newowner, author, quantity, memo)
 cancelofferf		(owner, [ftofferid1,...,ftofferidn])
 claimf			(claimer, [ftofferid1,...,ftofferidn])

 openf			(owner, author, symbol, ram_payer)
 closef			(owner, author, symbol)
 
```

# Data Structures  
## Assets  
```
sasset {  
	uint64_t	id; 		// asset id used for transfer and search;  
	name		owner;  	// asset owner (mutable - by owner!!!);  
	name		author;		// asset author (game contract, immutable);  
	name		category;	// asset category, chosen by author, immutable;  
	string		idata;		// immutable assets data. Can be stringified JSON or just sha256 string;  
	string		mdata;		// mutable assets data, added on creation or asset update by author. Can be  
					// stringified JSON or just sha256 string;  
	sasset[]	container;	// other NFTs attached to this asset
	account[]	containerf;	// FTs attached to this asset
}  
```
// Please include in idata or mdata info about asset name img desc which will be used by Markets  

## Offers  
```
offers {  
	uint64_t	assetid;	// asset id offered for claim ; 
	name		owner;		// asset owner;  
	name		offeredto;	// who can claim this asset ; 
	uint64_t	cdate;		// offer create date;  
}  
```

## Authors  
```
authors {  
	name	author;			// assets author, who will be able to create and update assets;  
	string	data;			// author’s data (json) will be used by markets for better display;  
	string	stemplate;		// data (json) schema for markets. key: state values, where key is key from;  
					// recommendations for non-text fields: hide, url, img, webgl, mp3, video;  
}  
```

## Delegates  
```
delegates{  
	uint64_t	assetid;	// asset id offered for claim;  
	name		owner;		// asset owner;  
	name		delegatedto;	// who can claim this asset;  
	uint64_t	cdate;		// offer create date;  
	uint64_t	period;		// Time in seconds that the asset will be lent. Lender cannot undelegate until 
					// the period expires, however the receiver can transfer back at any time.
}  
```

## Currency Stats (Fungible Token)
```
stat {  
	asset		supply;		// Tokens supply 
	asset		max_supply;	// Max token supply
	name		issuer;		// Fungible token author
	uint64_t	id;		// Unique ID for this token
	bool		authorctrl;	// if true(1) allow token author (and not just owner) to burn and transfer.
	string		data;		// strigified json. recommended keys to include: `img`, `name`
}
```

## Account (Fungible Token)  
```
accounts {  
	uint64_t	id;		// token id, from stat table
	asset		balance;	// token balance
}  
```

```
sofferf {
	uint64_t	id;		// id of the offer for claim (increments automatically) 
	name		author;		// ft author
	name		owner;		// ft owner
	asset		quantity;	// quantity
	name		offeredto;	// account who can claim the offer
	uint64_t	cdate;		// offer creation date
}
```    
  
# EXAMPLES: how to use Simple Assets in smart contracts

## Creating Asset and transfer to owner account ownerowner22:
```
name SIMPLEASSETSCONTRACT = "simpl5assets"_n;

name author = get_self();
name category = "weapon"_n;
name owner = "ownerowner22"_n;
string idata = "{\"power\": 10, \"speed\": 2.2, \"name\": \"Magic Sword\" }";
string mdata = "{\"color\": \"bluegold\", \"level\": 3, \"stamina\": 5, \"img\": \"https://bit.ly/2MYh8EA\" }";

action createAsset = action(
	permission_level{author, "active"_n},
	SIMPLEASSETSCONTRACT,
	"create"_n,
	std::make_tuple( author, category, owner, idata, mdata,	0 )
);
createAsset.send();	
```

## Creating Asset with requireclaim option for ownerowner22:
```
name SIMPLEASSETSCONTRACT = "simpl5assets"_n;

name author = get_self();
name category = "balls"_n;
name owner = "ownerowner22"_n;
string idata = "{\"radius\": 2, \"weigh\": 5, \"material\": \"rubber\", \"name\": \"Baseball\" }";
string mdata = "{\"color\": \"white\", \"decay\": 99, \"img\": \"https://i.imgur.com/QoTcosp.png\" }";


action createAsset = action(
	permission_level{author, "active"_n},
	SIMPLEASSETSCONTRACT,
	"create"_n,
	std::make_tuple( author, category, owner, idata, mdata, 1 )
);
createAsset.send();	
```

## Search asset and get assets info
1. Please add in your hpp file info about assets structure 
	**WARNING!!! CDT currently has a bug that doesn't allow compilation (v1.6.1).  
	1.5.0 also has a bug "Segmentation fault (core dumped)", but only with abi generation (including array of self objects : std::vector<sasset> container;)**
```
TABLE account {
	uint64_t	id;
	name		author;
	asset		balance;

	uint64_t primary_key()const { 
		return id;
	}
};
typedef eosio::multi_index< "accounts"_n, account > accounts;

TABLE sasset {
	uint64_t		id;
	name			owner;
	name			author;
	name			category;
	string			idata;
	string			mdata;
	std::vector<sasset>	container;
	std::vector<account>	containerf;

			
	auto primary_key() const {
		return id;
	}

	uint64_t by_author() const {
		return author.value;
	}
};

typedef eosio::multi_index< "sassets"_n, sasset, 		
		eosio::indexed_by< "author"_n, eosio::const_mem_fun<sasset, uint64_t, &sasset::by_author> >
> sassets;
```

2. Searching and using info
```
name SIMPLEASSETSCONTRACT = "simpl5assets"_n;
name author = get_self();
name owner = "lioninjungle"_n;

uint64_t assetid = 100000000000187

sassets assets(SIMPLEASSETSCONTRACT, owner.value);
auto idx = assets.find(assetid);

check(idx != assets.end(), "Asset not found or not yours");

check (idx->author == author, "Asset is not from this author");

auto idata = json::parse(idx->idata);  // for parsing json here is used nlohmann lib
auto mdata = json::parse(idx->mdata);  // https://github.com/nlohmann/json

check(mdata["cd"] < now(), "Not ready yet for usage");
```

## Update Asset
```
name SIMPLEASSETSCONTRACT = "simpl5assets"_n;

auto mdata = json::parse(idxp->mdata);
mdata["cd"] = now() + 84600;

name author = get_self();
name owner = "ownerowner22"_n;
uint64_t assetid = 100000000000187;

action saUpdate = action(
	permission_level{author, "active"_n},
	SIMPLEASSETSCONTRACT,
	"update"_n,
	std::make_tuple(author, owner, assetid, mdata.dump())
);
saUpdate.send();
```

## Transfer one Asset
```
name SIMPLEASSETSCONTRACT = "simpl5assets"_n;

name author = get_self();
name from = "lioninjungle"_n;
name to = "ohtigertiger"_n;

std::vector<uint64_t> assetids;
assetids.push_back(assetid);

string memo = "Transfer one asset";

action saUpdate = action(
	permission_level{author, "active"_n},
	SIMPLEASSETSCONTRACT,
	"transfer"_n,
	std::make_tuple(from, to, assetids, memo)
);
saUpdate.send();
```

## Transfer two Asset to same receiver with same memo  
```
name SIMPLEASSETSCONTRACT = "simpl5assets"_n;

name author = get_self();
name from = "lioninjungle"_n;
name to = "ohtigertiger"_n;

std::vector<uint64_t> assetids;
assetids.push_back(assetid1);
assetids.push_back(assetid2);

string memo = "Transfer two asset"

action saUpdate = action(
	permission_level{author, "active"_n},
	SIMPLEASSETSCONTRACT,
	"transfer"_n,
	std::make_tuple(from, to, assetids, memo)
);
saUpdate.send();
```

## issuef (fungible) issue created token
```
name SIMPLEASSETSCONTRACT = "simpl5assets"_n;

asset wood;
wood.amount = 100;
wood.symbol = symbol("WOOD", 0);

name author = get_self();
name to = "lioninjungle"_n;

std::string memo = "WOOD faucet";
action saRes1 = action(
	permission_level{author, "active"_n},
	SIMPLEASSETSCONTRACT,
	"issuef"_n,
	std::make_tuple(to, author, wood, memo)
);
saRes1.send();
```

## transferf (fungible) by author if authorctrl is enabled
```
name SIMPLEASSETSCONTRACT = "simpl5assets"_n;

asset wood;
wood.amount = 20;
wood.symbol = symbol("WOOD", 0);

name from = "lioninjungle"_n;
name to = get_self();
name author = get_self();

std::string memo = "best WOOD";
action saRes1 = action(
	permission_level{author, "active"_n},
	SIMPLEASSETSCONTRACT,
	"transferf"_n,
	std::make_tuple(from, to, author, wood, memo)
);
saRes1.send();
```

## burnf (fungible) by author if authorctrl is enabled
```
name SIMPLEASSETSCONTRACT = "simpl5assets"_n;

asset wood;
wood.amount = 20;
wood.symbol = symbol("WOOD", 0);

name author = get_self();
name from = "lioninjungle"_n;

std::string memo = "WOOD for oven";
action saRes1 = action(
	permission_level{author, "active"_n},
	SIMPLEASSETSCONTRACT,
	"burnf"_n,
	std::make_tuple(from, author, wood, memo)
);
saRes1.send();
```

