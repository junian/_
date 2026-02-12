# Invoice Radar

```bash
npx asar extract "/Applications/Invoice Radar.app/Contents/Resources/app.asar" ~/Documents/invoice-radar-re/
```

Need to decrypt following:

```bash
```

Find `authTag`.

Found out possibly decryption function:

```javascript
IVr(r.encrypted,r.iv,r.authTag,A)
```

We already have the `encrypted`, `iv`, and `authTag` from the JSON API above.
But what about `A`?
Seems like `A` is the key to decrypt, search for it and got from:

```javascript
const A=yield*t.getSharedSecret()
```
Found out the `getSharedSecret`:

```javascript
getSharedSecret:()=>we(function*(){return yield*D0(async()=>{const t=await DVr(),n=await Swt.load("co_zmPyjq18WXXURamZmtPoSgjFP9D",{loadAs:t});if(n?.$isLoaded)return n?.latestBuildHash})})
```

Let's format it:

```javascript
getSharedSecret: () => we(function* () { 
    return yield* D0(async () => { 
        const t = await DVr(), 
        n = await Swt.load("co_zmPyjq18WXXURamZmtPoSgjFP9D", { loadAs: t }); 
        if (n?.$isLoaded) return n?.latestBuildHash 
    }) 
});
```

Let's find out what `DVr()` is:

<!-- uncensored

```javascript
const DVr=async()=>(await kwt({syncServer:"wss://sync.invoiceradar.com",accountID:"co_zQBgweBL2DpK9sMFbtaLpnnqrXK",accountSecret:"sealerSecret_zEbvA7ZnscxmxRov6ZHPng3VBq2xLsdWDU768UiQJbEL/signerSecret_zEmTuwhUQmxhXvmimEaEtb3RZ4GKG1g3oAg4dEMjpb1AM"}))
```

-->

```javascript
const DVr=async()=>(await kwt({syncServer:"wss://sync.****.com",accountID:"co_****",accountSecret:"sealerSecret_****/signerSecret_****"}));
```

Looks like we hit the jackpot where the app was getting the key.

By the pattern of it, it's using the distributed database called [Jazz][jazz-tools].

Let's try accessing the Jazz DB by using the [Jazz Inspector][jazz-inspector].
Use the `syncServer`, `accountID`, and `accountSecret` to login.



[jazz-tools]: https://jazz.tools/
[jazz-inspector]: https://inspector.jazz.tools/
