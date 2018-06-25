# Thor Java Client SDK
- - - -
A SDK for client toolkit to call Restful API.

## License
Thor Java Client SDK is licensed under the GNU Lesser General Public License v3.0, also included in LICENSE file in repository.
- - - -

## Set blockchain nodes provider
Set blockchain nodes sample as follows

```
NodeProvider nodeProvider = NodeProvider.getNodeProvider();
nodeProvider.setProvider("http://blockchain-nodes-url");
nodeProvider.setTimeout(10000);
```

##  You can find the clients toolkit under the directory :
 **src/main/java/com/vechain/thorclients/clients**

And also there is the detail documents <a href="https://github.com/vechain/thor-client-sdk4j/blob/dev/doc"> sdk4j doc </a>

### AccountClient
User can use this client :
- Get Account information: VET balance and VTHO balance
```
Address address = Address.fromHexString(fromAddress);
Account account = AccountClient.getAccountInfo(address, null);
logger.info("account info:" + JSON.toJSONString(account));
logger.info("VET:" + account.VETBalance().getAmount() + " Energy:" + account.energyBalance().getAmount());
Assert.assertNotNull(account);

```
- Call Contract view method.
```
Address contractAddr = token.getContractAddress();
Revision currRevision = revision;
if(currRevision == null){
    currRevision = Revision.BEST;
}
AbiDefinition abiDefinition = ERC20Contract.defaultERC20Contract.findAbiDefinition("balanceOf");
ContractCall call = ERC20Contract.buildCall( abiDefinition, address.toHexString( null ) );
ContractCallResult contractCallResult = callContract(call, contractAddr,  currRevision );

```
- Get code on a address.
```
Address tokenAddr = Address.VTHO_Address;
AccountCode code = AccountClient.getAccountCode(tokenAddr, null);
logger.info("code:" + JSON.toJSONString(code));

```

- - - -
### TransactionClient
- Send VET to account
```
byte chainTag = BlockchainClient.getChainTag();
byte[] blockRef = BlockchainClient.getBlockRef( Revision.BEST).toByteArray();
Amount amount = Amount.createFromToken( AbstractToken.VET);
amount.setDecimalAmount( "21.12" );
ToClause clause = TransactionClient.buildVETToClause(
        Address.fromHexString( "VXc71ADC46c5891a8963Ea5A5eeAF578E0A2959779" ),
        amount,
        ToData.ZERO );
RawTransaction rawTransaction =RawTransactionFactory.getInstance().createRawTransaction( chainTag, blockRef, 720, 21000, (byte)0x01, CryptoUtils.generateTxNonce(), clause);
TransferResult result = TransactionClient.signThenTransfer( rawTransaction, ECKeyPair.create( privateKey ) );
logger.info( "transfer vet result:" + JSON.toJSONString( result ) );

```
- Send VTHO to account
```
byte chainTag = BlockchainClient.getChainTag();
byte[] blockRef = BlockClient.getBlock( null ).blockRef().toByteArray();
Amount amount = Amount.createFromToken( ERC20Token.VTHO);
amount.setDecimalAmount( "11.12" );
ToClause clause = ERC20Contract.buildTranferToClause( ERC20Token.VTHO,
        Address.fromHexString("VXc71ADC46c5891a8963Ea5A5eeAF578E0A2959779"),
        amount);
RawTransaction rawTransaction =RawTransactionFactory.getInstance().createRawTransaction( chainTag, blockRef, 720, 80000, (byte)0x01, CryptoUtils.generateTxNonce(), clause);

TransferResult result = TransactionClient.signThenTransfer( rawTransaction, ECKeyPair.create( privateKey ) );
logger.info( "transfer vethor result:" + JSON.toJSONString( result ) );

```
- Query transaction receipt
```
Receipt receipt = TransactionClient.getTransactionReceipt(setUserPlanTxId, null);
logger.info("Receipt:" + JSON.toJSONString(receipt));

```
- Query transaction
```
Transaction transaction = TransactionClient.getTransaction(hexId, true, null);
logger.info("Transaction:" + JSON.toJSONString(transaction));

```

- - - -
### BlockClient
You can get block by specified the block revision.
```
Transaction transaction = TransactionClient.getTransaction(hexId, false, null);
logger.info("Transaction:" + JSON.toJSONString(transaction));

```

- - - -
### LogsClient
You can get events logs and transfer logs, the api is also supporting pagination query.
- Query events logs.
```
EventFilter filter = EventFilter.createFilter( Range.createBlockRange(1000, 20000), Options.create( 0, 10 ) );
ArrayList filteredEvents =  LogsClient.filterEvents( filter, Order.DESC, null);

```
- Query transfer logs.
```
TransferFilter filter = TransferFilter.createFilter(Range.createBlockRange( 1000, 20000 ) ,Options.create( 0, 10 ) );
ArrayList transferLogs = LogsClient.filterTransferLogs( filter, Order.DESC);
```

- - - -
### BlockchainClient
You can get the chain tag and block reference.
- Get chain tag
```
byte chainTag = BlockchainClient.getChainTag();
int chainTagInt = chainTag & 0xff;
logger.info( "chainTag: " + chainTagInt);
```
- Get block reference
```
Revision revision = Revision.create(0);
Block block = BlockClient.getBlock(revision);
logger.info("blockRef;" + BytesUtils.toHexString(block.blockRef().toByteArray(), Prefix.ZeroLowerX));
```


- - - -
### ProtoTypeClient
The detail information you can refer to the page[ProtoType Wiki](https://github.com/vechain/thor/wiki/Prototype(CN))
- Get master address 
```
ContractCallResult callResult = ProtoTypeContractClient.getMasterAddress( Address.fromHexString( fromAddress ) , Revision.**BEST**);
logger.info( "testGetMaster result:" + JSON.toJSONString( callResult ) );

```

- Set master address
```
TransferResult result = ProtoTypeContractClient.setMasterAddress( new Address[]{Address.fromHexString( fromAddress ) }, new Address[]{Address.fromHexString( fromAddress )},ContractClient.GasLimit, (byte)0x1, 720, ECKeyPair.create(privateKey ) );
logger.info( "result: " + JSON.toJSONString( result ) );

```

- Add user
```
TransferResult transferResult = ProtoTypeContractClient.addUser(
        new Address[]{Address.fromHexString( fromAddress )},
        new Address[]{Address.fromHexString(UserAddress)},
        ContractClient.**GasLimit**, (byte)0x1, 720, ECKeyPair.create( privateKey ) );
logger.info("Add user:" + JSON.toJSONString( transferResult ));

```

- Check if it is user
```
ContractCallResult callResult = ProtoTypeContractClient.isUser( Address.fromHexString( fromAddress ) ,Address.fromHexString( UserAddress),
        Revision.BEST);
logger.info( "Get isUser result:" + JSON.toJSONString( callResult ) );
```
- Remove user
```
TransferResult transferResult = ProtoTypeContractClient.removeUsers(
      new Address[]{Address.fromHexString( fromAddress )},
      new Address[]{Address.fromHexString( UserAddress)},
        ContractClient.GasLimit, (byte)0x1, 720, ECKeyPair.create( privateKey ) );
logger.info( "Remove user:"  + JSON.toJSONString( transferResult ));

```
- Set User plan
```
Amount credit = Amount.VTHO();
credit.setDecimalAmount( "12.00" );
Amount recovery = Amount.VTHO();
recovery.setDecimalAmount( "0.00001" );

TransferResult result = ProtoTypeContractClient.setUserPlans(
        new Address[]{Address.fromHexString( fromAddress)},
        new Amount[]{credit},
        new Amount[]{recovery},
        ContractClient.GasLimit, (byte)0x1, 720, ECKeyPair.create( privateKey ) );
logger.info( "set user plans:" + JSON.toJSONString( result ) );

```
- Get User plan
```
ContractCallResult callResult = ProtoTypeContractClient.getUserPlan( Address.fromHexString( fromAddress ) , Revision.BEST);
logger.info( "Get user plan result:" + JSON.toJSONString( callResult ) );

```
- Get User credits
```
ContractCallResult callResult = ProtoTypeContractClient.getUserCredit(
        Address.fromHexString( fromAddress ),
        Address.fromHexString( UserAddress),
        Revision.BEST);
logger.info( "Get user plan result:" + JSON.toJSONString( callResult ) );

```




- - - -
### Run the test.
Some case may be failed because of the account or block is not existed on your blockchain env.

```
mvn clean install
```


### 4 Java console approach

The SDK support the command line approach to get chainTag, blockRef, create wallet, sign transaction; Run with maven：

```
mvn clean package

The maven will generate the jar file in folder target: thor-client-sdk4j-0.0.2.jar
```

```
Run the following command:

There is a example transaction file in src/main/resources/exchange_example.xlsx
 
- Get chainTag: java -jar thor-client-sdk4j-0.0.2.jar chainTag {blockchain-server-url}
  ChainTag:
  0x9a
  
- Get blockRef: java -jar thor-client-sdk4j-0.0.2.jar blockRef {blockchain-server-url}
  BlockRef:
  0x000245e360d4cd1b
  
- Create wallet: java -jar thor-client-sdk4j-0.0.2.jar createWallet {wallet-password}
  The wallet created successfully and the key store is:
  {"address":"0xc283b29b9e46ec62b1cc78d95ad2598e58f8af17","crypto":{"cipher":"aes-128-ctr","cipherparams":{"iv":"8bd06ab3901ec874c35c396736624800"},"ciphertext":"f69d08d62cdadae76da47e76e6d59b7c60e8e2bb8919c785bac0f6f8621ddfed","kdf":"scrypt","kdfparams":{"dklen":32,"n":262144,"p":1,"r":8,"salt":"b4e5b39cadcb66f4682f7863dde854b4a87b3810864d1ab6fafd1cdc923a2a1c"},"mac":"a2ee21d526a1d0113fde864bf67c55ff2b2cd85fc6a8f232e21d3b59da9790b3"},"id":"656a8193-a670-48fc-bbf7-eedbdb2dd5e7","version":3}
  The wallet created successfully and the privateKey is:
  0x5eabfe97a3854a16b2194ff18baf14471809896c73627414c7fbd35bd7431014
  
- Sign transactions: java -jar thor-client-sdk4j-0.0.2.jar sign {your-file-path} {privateKey}
  Raw Transaction:
  0xf902d6819a8702288058b9af928202d0f90273e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f8000080e094d3ef28df6b553ed2fc47259e8134319cb1121a2a89364200111c48f800008001830616988088ff9198c817655decc0b841bd61e198f126adddb169eebf5cd3da25ae3a3f07102e574bcd1368440d1e307c4c47884364e2abc66ef6940c4953758dd1c57f8255025639702104ce83e9a3b501
- Send transactions: java -jar thor-client-sdk4j-0.0.2.jar send {blockchain-server-url} {privateKey} {your-file-path}
  
```

