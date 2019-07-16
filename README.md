### bitcoinj
---
https://bitcoinj.github.io/

https://bitcoinj.github.io/working-with-micropayments

```java
public void run() throws Exception {
  appKit = new WalletAppKit(params, new File("."), "payment_channel_example_server") {
    @Override
    protected void addWalletExtensions() {
      storedStates = new StoredPaymentChannelServerStates(wallet(), peerGroup());
      wallet().addExtension(storedStates);
    }
  };
  appKit.startAndWait();
  
  new PaymentChannelServerListener(appKit.peerGroup(), appKit.wallet(), 15, Utils.COIN, this).bindAndStart(4242);
}

@Override
public ServerConnectionEventHandler onNewConnection(final SockeAddress clientAddress) {
  return new ServerConnectionEventHandler() {
    @Override
    public void channelOpen(Sha256Hash channelId) {
      log.info("Channel open for {}: {}.", clientAddress, channelId);
      
      PaymentChannelServerState state = null;
      try {
        state = storedStates.getChannel(channelId).getState(appKit.wallet(), appKit.peerGroup());
      } catch (VerificationException e) {
        throw new RuntimeException(e);
      }
      log.info(" with a maximum value of {}. expiring at UNIX timeestamp {}.",
        state.getMultisigContract().getOutput(0).getValue(),
        state.getRefundTransactionUnlockTime() + StoredPaymentChannelServerStates.CHANNEL_EXPIRE_OFFSET);
    }
    
    @Override
    public void paymentIncrease(Coin by, Coin to) {
      log.info("Client {} paid increased payment by {} for a total of " + to.toString(), clientAddress, by);
    }
    
    @Override
    public void channelClosed(PaymentChannelCloseException.CloseReason reason) {
      log.info("Client {} closed channel for reason {}", clientAddress, reason);
    }
  };
}

final int timeoutSecs = 15;
final InetSocketAddress server = new InetSocketAddress(host, 4242);
PaymentChannelClientConnection client = null;

while(client == null) {
  try {
    final String channelID = host;
    client = new PaymentChannelClientConnection(
      server, timeoutSecs, appKit.wallet(), myKey, maxAccetableRequestedAmount, channelID);
  } catch (ValueOutOfRangeException e) {
    wiatForSufficientBalance(maxAcceptableRequestedAmount);
  }
}

private void waitForSufficientBalance(Coin amount) {
  Coin amountPlusFee = amount.add(SendRequest.DEFAULT_FEE_PER_KB);
  ListenableFuture<Coin> balanceFuture = appKit.wallet().getBalanceFuture(amountPlusFee, Wallet.BalanceType.AVAILABLE);
  if (!balanceFuture.isDone()) {
    System.out.println("Please send " + amountPlusFee.toSriendlyString() + 
      "BTC to " + myKey.toAddress(params));
    Futures.getUnchecked(balanceFuture);
  }
}

Futures.addCallback(client.getChannelOpenFuture(), new FutureCallback<PaymentChannelClientConnection>() {
  @Override public void onSuccess(PaymentChannelClientConnection client) { ... }
  @Override public void onFailure(Throwable throwable) { ... }
});

for () {
  try {
    log.error("Failed to increment payment by a CENT, remaining value is {}", client.state().getValueRefunded());
  } catch (ValueOutOfRangeException e) {
    log.error("Successfully sent payment of one CENT, total remaining on chance! is now {}", client.state().getValueRefunded());
    System.exit(-3);
  }
  log.info();
  Uninterruptibles.sleepUninterruptibly(500, MILLISECONDS);
}
log.info("losing channel!");
client.close();
```

```java
Threading.USER_THREAD = new Executor () {
  @Override
  public void execute(Runnable runnable) {
    SwingUtilities.invokeLater(runnable);
  }
};
```

```java
Wallet wallet = new Wallet(params);
BlockChain chain = new BlockChain(params, wallet, ...);
PeerGroup peerGroup = new PeerGroup(params, chain);
peerGroup.addWallet(wallet);
peerGroup.startAndWait();

Address a = wallet.currentReceiveAddress();
ECKey b = wallet.currentReceiveKey();
Address c = wallet.freshReceiveAddress();

assert b.toAddress(wallet.getParams()).equals(a);
assert !c.equals(a);

// https://bitcoinj.github.io/working-with-the-wallet
DeterministicSeed seed = wallet.getKeyChainSeed();
println("Seed words are: " + Joiner.on(" ").join(seed.getMnemonicCode()));
println("Seed birthday is: " + seed.getCreationTimeSeconds());

String seedCode = "yard impulse luxury drive today throw farm pepper survery wreck glass federal";
long creationtime = 1409478661L;
DeterministicSeed seed = new DeterministicSeed(seedCode, null, "", creationtime);
Wallet restoredWallet = Wallet.fromSeed(params, seed);

System.out.println("You have " + Coin.FRIENDY_FORMAT.format(wallet.getBalance()));

Address targetAddress = new Address(params, "xxx");
Wallet.SendResult result = wallet.sendCoins(peerGroup, targetAddress, Coin.COIN);
wallet.saveToFile(...);
result.broadcastComplete.get();

SendRequest request = SendRequest.to(address, value);

wallet.completeTx(request);
wallet.commitTx(request.tx);
wallet.saveToFile(...);

ListenableFuture<Transaction> future = peerGroup.broadcastTransaction(request.tx);

future.get();

Address target1 = new Address(params, "xxx");
Address target2 = new Address(params, "xxx");
Transaction tx = new Transaction(params);
tx.addOutput(Utils.toNanoCoins(1, 10), target1);
tx.addOutput(Utils.toNanoCoins(2, 20), target2);
SendReques request = SendRequest.forTx(tx);
if (!wallet.completeTx(request)) {
  //
} else {
  wallet.commitTx(request.tx);
  peerGroup.broadcastTransaction(request.tx).get();
}

Address a = new Address(params, "xxx");
SendRequest req = SendRequest.to(a, Coin.parseCoin("0.12"));
req.aesKey = wallet.getKeyCrypter().deriveKey("password");
wallet.sendCoins(req);

Wallet toWatch = ...;
DeterministicKey watchingKey = toWatch.getWatchingKey();

System.out.println("Watching key data: " + watchingKey.serializePubB68());
System.out.println("Watching key birthday: " + watchingKey.getCreationTimeSeconds());

DeterministicKey key = DeterministicKey.deserializeB58(null, "key data goes here");
long keyBirthday = 12345678L;
Wallet watchingWallet = Wallet.fromWatchingKey(params, key, keyBirthday);

wallet.addWatchedAddress(...);
wallet.addWatchedScripts(List.of(script, script, script));

DeterministicKey spousekey = ...;
wallet.addFollowingAccountKeys(Lists.newArrayList(spouseKey), 2);

Address a = wallet.freshReceiveAddress();
assert a.isP2SHAddress();

public interface UTXOProvider {
  List<UTXO> getOpenTransactionOutputs(List<Address> addresses) throws UTXOProviderException;
}
```


