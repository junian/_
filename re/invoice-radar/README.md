# Invoice Radar

```bash
npx asar extract "/Applications/Invoice Radar.app/Contents/Resources/app.asar" ~/Documents/invoice-radar-re/
```

Need to decrypt following:

<!-- 
```
https://invoiceradar.com/plugins.json?v=2&c=1
```
-->

```bash
https://****.com/plugins.json?v=2&c=1
```

Find `authTag`.

Found out possibly decryption function:

```javascript
IVr(r.encrypted,r.iv,r.authTag,A)
```

Let's find find out the `IVr` function, found this one:

```bash
const mVr="aes-256-gcm";function IVr(e,t,n,r){const i=wl.createDecipheriv(mVr,Buffer.from(r,"hex"),Buffer.from(t,"base64"));i.setAuthTag(Buffer.from(n,"base64"));const s=Buffer.concat([i.update(Buffer.from(e,"base64")),i.final()]);return JSON.parse(s.toString("utf8"))}
```

Let's beautify it:

```javascript
const mVr = "aes-256-gcm";
function IVr(e, t, n, r) { 
    const i = wl.createDecipheriv(mVr, Buffer.from(r, "hex"), Buffer.from(t, "base64")); 
    i.setAuthTag(Buffer.from(n, "base64")); 
    const s = Buffer.concat([i.update(Buffer.from(e, "base64")), i.final()]); 
    return JSON.parse(s.toString("utf8")) 
}
```

From the look of it, it seems like a standard `aes-256-gcm` encryption.
We can reconstruct it using the standard Node.js `crypto` package.

Now let's go back to `IVr` execution. Take a look at the parameters.
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
Use the key from `Swt.load` to get the CoValue.

Got this value.
Based on `getSharedSecret`, it's using the value of `latestBuildHash` property.

<!-- uncensored
```bash
latestBuildHash
3e03c49d674d226b3057f360fa9dbd296fbce0177ace1703f9ecf5f2594aaab2
```
-->

```bash
latestBuildHash
3e03***********
```

[jazz-tools]: https://jazz.tools/
[jazz-inspector]: https://inspector.jazz.tools/
