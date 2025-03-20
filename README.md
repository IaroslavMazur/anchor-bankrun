# anchor-bankrun

`anchor-bankrun` is a small but powerful extension to [solana-bankrun](https://github.com/kevinheavey/solana-bankrun)
that enables using both Anchor and Bankrun with only a one-line code change. It does this by exporting a `BankrunProvider` class that can be used as a drop-in replacement for `AnchorProvider` during testing.

## Anchor version note

Recent versions of `anchor-bankrun` use the Anchor v0.31 IDL, which is not backwards compatible with older Anchor IDLs.
If you have an older IDL, use `anchor-bankrun` v0.3.0.

## Note on the incompatibility with RPC wrappers

Because [Bankrun](https://github.com/kevinheavey/solana-bankrun/tree/main) doesn't include an RPC system (and, as a result, the `BankrunProvider.connection` field is empty), the libraries that require a valid `Connection` to be passed to their functions (such as the [@solana/spl-token](https://www.npmjs.com/package/@solana/spl-token) library) will not work with `anchor-bankrun`.

As an alternative, you can use the [spl-token-bankrun](https://www.npmjs.com/package/spl-token-bankrun) library which provides wrappers for the [@solana/spl-token](https://www.npmjs.com/package/@solana/spl-token) functions (such as `createMint()` and `mintTo()`).

## Usage

Here's an example using `BankrunProvider` to test an Anchor program:

```typescript
import { BankrunProvider, startAnchor } from "anchor-bankrun";
import { Keypair, PublicKey } from "@solana/web3.js";
import { BN, Program } from "@coral-xyz/anchor";
import { Puppet } from "./anchor-example/puppet";
const IDL = require("./anchor-example/puppet.json");

test("anchor", async () => {
  const context = await startAnchor("tests/anchor-example", [], []);

  const provider = new BankrunProvider(context);

  const puppetProgram = new Program<Puppet>(IDL, provider);

  const puppetKeypair = Keypair.generate();
  await puppetProgram.methods
    .initialize()
    .accounts({
      puppet: puppetKeypair.publicKey,
    })
    .signers([puppetKeypair])
    .rpc();

  const data = new BN(123456);
  await puppetProgram.methods
    .setData(data)
    .accounts({
      puppet: puppetKeypair.publicKey,
    })
    .rpc();

  const dataAccount = await puppetProgram.account.data.fetch(
    puppetKeypair.publicKey
  );
  expect(dataAccount.data.eq(new BN(123456)));
});
```

## Installation

```
yarn add anchor-bankrun
```

## Why is this a separate package?

I want to keep the Bankrun dependencies light.
