# Gas and Fees消耗理解

gas规则
Gas 和 Fees
可以使用这几个参数来表明他们愿意支付多少fee：

> --gas 指的是gas的数量，gas代表Tx消耗的计算资源，需要消耗多少gas取决于具体的Tx，在Tx执行之前无法被精确计算出来，但可以通过在--gas后带上参数auto来进行估算。

> --gas-adjustment（可选）可用于适当的增加 gas ，以避免其被低估。例如，用户可以将gas-adjustment设为1.5，那么被指定的gas将是被估算gas的1.5倍。如果手动设置了gas限制，则该标志将被忽略（默认为1）

> --gas-prices 指定用户愿意为每单位gas支付多少fee，可以是一种或多种代币。例如，--gas-prices=0.025uatom, 0.025upho就表明用户愿意为每单位的gas支付0.025uatom 和 0.025upho。

> --fees 指定用户总共愿意支付的fee。

> --fee-payer 选项指定一个不同的账户来支付交易的手续费

> --fee-granter 来授权第三方为其支付手续费


支付fee的最终价值等于gas的数量乘以gas的价格。换句话说，fees = ceil(gas * gasPrices)。由于可以使用gas价格来计算fee，也可以使用fee来计算gas价格，因此用户仅指定两者之一即可。
随后，验证者通过将给定的或计算出的gas-prices与他们本地的min-gas-prices进行比较，来决定是否在其区块中写入该Tx。如果gas-prices不够高，该Tx将被拒绝，因此鼓励用户支付更多fee。


min-gas-prices价格由网络的每个验证器设置。验证者将评估他们的利润、区块链的稳定性和相关因素来设定最低但可接受的 gas 价格，理论上gas价格可以设置为0

> 在app.toml下面配置最小 minimum-gas-prices = "1stake"

> ./mychaind tx orders generate-order 10.21.3.28 my_pub_rsa aliyun 10000stake --home /Users/zuiyou/zuiyou_work/mychain/cmd/node1  --from cosmos1hwkqjfdqamns0r68ek50fperqh452vlam2yqmc --gas=200000 --gas-prices 0.1stake
> raw_log: 'insufficient fees; got: 20000stake required: 200000stake: insufficient fee'
> 最小：200000*1stake 只支付200000*0.1stake 小于最低支付gas导致的失败

> ./mychaind tx orders generate-order 10.21.3.28 my_pub_rsa aliyun 10000stake --home /Users/zuiyou/zuiyou_work/mychain/cmd/node1  --from cosmos1hwkqjfdqamns0r68ek50fperqh452vlam2yqmc --gas=200 --gas-prices 1stake
> raw_log: 'out of gas in location: ReadFlat; gasWanted: 200, gasUsed: 1000: out of gas' gas_used: "1000" gas_wanted: "200"
> 当你设置的最大gas小于实际的gas消耗导致的失败

> ./mychaind tx orders generate-order 10.21.3.28 my_pub_rsa aliyun 10000stake --home /Users/zuiyou/zuiyou_work/mychain/cmd/node1  --from cosmos1hwkqjfdqamns0r68ek50fperqh452vlam2yqmc --fees 1stake 
> raw_log: 'insufficient fees; got: 1stake required: 200000stake: insufficient fee'
> 当你用fee时且不指定gas时，会默认用200000*min-gas-prices = 200000stake 作为你要支付的费用，导致失败

> ./mychaind tx orders generate-order 10.21.3.28 my_pub_rsa aliyun 10000stake --home /Users/zuiyou/zuiyou_work/mychain/cmd/node1  --from cosmos1hwkqjfdqamns0r68ek50fperqh452vlam2yqmc --fees 1stake --gas 20000
> raw_log: 'insufficient fees; got: 1stake required: 200000stake: insufficient fee'
> 当你用fee时指定gas时，会用指定的20000*min-gas-prices = 20000stake 作为你要支付的费用，导致失败

> ./mychaind tx orders generate-order 10.21.3.28 my_pub_rsa aliyun 10000stake --home /Users/zuiyou/zuiyou_work/mychain/cmd/node1  --from cosmos1d0zfjyzdzs7trndn9dpnzuwennrca69xsne3n0 --fees 50000stake --gas-prices 1stake
>  cannot provide both fees and gas prices

> ./mychaind tx orders generate-order 10.21.3.28 my_pub_rsa aliyun 10000stake --home /Users/zuiyou/zuiyou_work/mychain/cmd/node1  --from cosmos1d0zfjyzdzs7trndn9dpnzuwennrca69xsne3n0 --gas-prices 1stake

balances:
- amount: "98200000"
  denom: stake
- amount: "10000"
  denom: token
  pagination:
  next_key: null
  total: "0"





./mychaind q bank balances cosmos1d0zfjyzdzs7trndn9dpnzuwennrca69xsne3n0 --home /Users/zuiyou/zuiyou_work/mychain/cmd/node1


gas wanted 交易中指定的燃气上限，gas used表示实际使用的燃气量。
gas wanted 就是指定的--gas 如果不指定的话，在genesis.json第一次启动时配置，默认200000

##  提交交易的gas消耗计算规则
预估gas,但这个预估的比实际的要大很多
> ./mychaind tx orders generate-order 10.21.3.28 my_pub_rsa aliyun  10000stake --home /Users/zuiyou/zuiyou_work/mychain/cmd/node1  --from cosmos1hwkqjfdqamns0r68ek50fperqh452vlam2yqmc --gas=200000 --gas-prices 1stake --dry-run

当指定--gas时就一定要指定--gas-prices
也可以直接用--fees的方式指定 


gas具体的计算消耗由 数据库操作计算大小与msg的大小决定
--BroadcastTxCommit
    --CheckTx
    --DeliverTx
        --func (app *BaseApp) runTx(mode runTxMode, txBytes []byte)
            --AnteHandlers
                ConsumeTxSizeGasDecorator
                --anteHandler具体计算块的大小 ：ctx.GasMeter().ConsumeGas(params.TxSizeCostPerByte*sdk.Gas(len(ctx.TxBytes())), "txSize")
                NewSigGasConsumeDecorator
                --签名消耗的gas
            --app.runMsgs(runMsgCtx, msgs, mode) 具体操作消耗的gas
            
签名消耗gas
```go
func DefaultSigVerificationGasConsumer(
	meter sdk.GasMeter, sig signing.SignatureV2, params types.Params,
) error {
	pubkey := sig.PubKey
	switch pubkey := pubkey.(type) {
	case *ed25519.PubKey:
		meter.ConsumeGas(params.SigVerifyCostED25519, "ante verify: ed25519")
		return sdkerrors.Wrap(sdkerrors.ErrInvalidPubKey, "ED25519 public keys are unsupported")

	case *secp256k1.PubKey:
		meter.ConsumeGas(params.SigVerifyCostSecp256k1, "ante verify: secp256k1")
		return nil

	case *secp256r1.PubKey:
		meter.ConsumeGas(params.SigVerifyCostSecp256r1(), "ante verify: secp256r1")
		return nil

	case multisig.PubKey:
		multisignature, ok := sig.Data.(*signing.MultiSignatureData)
		if !ok {
			return fmt.Errorf("expected %T, got, %T", &signing.MultiSignatureData{}, sig.Data)
		}
		err := ConsumeMultisignatureVerificationGas(meter, multisignature, pubkey, params, sig.Sequence)
		if err != nil {
			return err
		}
		return nil

	default:
		return sdkerrors.Wrapf(sdkerrors.ErrInvalidPubKey, "unrecognized public key type: %T", pubkey)
	}
}
```

tx大小消耗的gas
```go
func (cgts ConsumeTxSizeGasDecorator) AnteHandle(ctx sdk.Context, tx sdk.Tx, simulate bool, next sdk.AnteHandler) (sdk.Context, error) {
	sigTx, ok := tx.(authsigning.SigVerifiableTx)
	if !ok {
		return ctx, sdkerrors.Wrap(sdkerrors.ErrTxDecode, "invalid tx type")
	}
	params := cgts.ak.GetParams(ctx)

	ctx.GasMeter().ConsumeGas(params.TxSizeCostPerByte*sdk.Gas(len(ctx.TxBytes())), "txSize")

	// simulate gas cost for signatures in simulate mode
	if simulate {
		// in simulate mode, each element should be a nil signature
		sigs, err := sigTx.GetSignaturesV2()
		if err != nil {
			return ctx, err
		}
		n := len(sigs)

		for i, signer := range sigTx.GetSigners() {
			// if signature is already filled in, no need to simulate gas cost
			if i < n && !isIncompleteSignature(sigs[i].Data) {
				continue
			}

			var pubkey cryptotypes.PubKey

			acc := cgts.ak.GetAccount(ctx, signer)

			// use placeholder simSecp256k1Pubkey if sig is nil
			if acc == nil || acc.GetPubKey() == nil {
				pubkey = simSecp256k1Pubkey
			} else {
				pubkey = acc.GetPubKey()
			}

			// use stdsignature to mock the size of a full signature
			simSig := legacytx.StdSignature{ //nolint:staticcheck // this will be removed when proto is ready
				Signature: simSecp256k1Sig[:],
				PubKey:    pubkey,
			}

			sigBz := legacy.Cdc.MustMarshal(simSig)
			cost := sdk.Gas(len(sigBz) + 6)

			// If the pubkey is a multi-signature pubkey, then we estimate for the maximum
			// number of signers.
			if _, ok := pubkey.(*multisig.LegacyAminoPubKey); ok {
				cost *= params.TxSigLimit
			}

			ctx.GasMeter().ConsumeGas(params.TxSizeCostPerByte*cost, "txSize")
		}
	}

	return next(ctx, tx, simulate)

```


gasConfig
    HasCost: 判断某个键是否存在的 Gas 消耗。
    DeleteCost: 删除某个键的 Gas 消耗。
    ReadCostFlat: 读取一条记录的 Gas 消耗（不包括数据）。
    ReadCostPerByte: 每读取一个字节的 Gas 消耗。
    WriteCostFlat: 写入一条记录的 Gas 消耗（不包括数据）。
    WriteCostPerByte: 每写入一个字节的 Gas 消耗。
    IterNextCostFlat: 迭代器向前移动一条记录的 Gas 消耗（不包括数据）。

具体一个存储所消耗的gas
```go
func (gs *Store) Set(key, value []byte) {
	types.AssertValidKey(key)
	types.AssertValidValue(value)
	gs.gasMeter.ConsumeGas(gs.gasConfig.WriteCostFlat, types.GasWriteCostFlatDesc)
	gs.gasMeter.ConsumeGas(gs.gasConfig.WriteCostPerByte*types.Gas(len(key)), types.GasWritePerByteDesc)
	gs.gasMeter.ConsumeGas(gs.gasConfig.WriteCostPerByte*types.Gas(len(value)), types.GasWritePerByteDesc)
	gs.parent.Set(key, value)
}
```

Main Gas Meter 主燃气表
跟踪导致状态转换的执行过程中消耗的气体。它在每次交易执行时重置。

Block Gas Meter 块燃气表

Gas退款
在以太坊中，gas 可以在执行前指定，如果有任何 gas 剩余，剩余的 gas 将被退还给用户——如果没有提供足够的 gas，应该会因 gas 耗尽而失败。
cosmos 中，不存在 gas 退款的概念，所支付的费用不会部分退还给用户。交易收取的费用将由验证者收取，并且不予退款。因此，使用正确的气体极为重要。
为防止费用超支，--gas-adjustment为 cosmos 交易提供标志将自动确定费用。

15676
16718
17760

##  使用双代币作为gas消耗


在Cosmos中，计算gas费用的方法是基于操作的执行成本。每个操作都有一个预设的gas费用，这个费用是由操作的执行成本所决定的。当你在Cosmos网络上发送一笔交易时，交易中包含的每个操作都会消耗一定数量的gas。

具体来说，Cosmos中每个操作的gas费用是由以下因素决定的：

操作的复杂度：操作的复杂度越高，所需的计算资源和时间就越多，因此需要支付更多的gas费用。

网络拥堵情况：当网络拥堵时，gas费用会上涨，以便更快地处理交易。反之，当网络空闲时，gas费用会降低，以便更便宜地处理交易。

市场供需关系：与其他加密货币一样，Atom代币的价格在市场上波动，这也会影响gas费用的计算。

在Cosmos中，gas费用是以Atom代币的形式支付的。因此，如果你想在Cosmos网络上发送交易，你需要确保你的钱包中有足够的Atom代币来支付gas费用。你可以使用Cosmos钱包中的gas计算器来估算交易的gas费用，并确定是否有足够的代币来支付这些费用