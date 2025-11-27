**Idea**
Patch the exploit.py which exploits parameters because they arent validated

**Solution**
Replace ```_eval()``` with 
```
function parseQuantity(input) {
  const n = Number(input);
  if (!Number.isFinite(n) || n < 0) return NaN;
  return n;
}
```

Update 
```function buildCartComputation(cart) {
  const ids = Object.keys(cart).map(Number).filter(Number.isFinite);
  const items = getItemsByIds(ids).sort((a, b) => a.id - b.id);

  const lines = items.map((it) => {
    const raw = String(cart[it.id] ?? "0");
    const qtyNum = Number(raw);
    const lineTotal = Number.isFinite(qtyNum) ? qtyNum * it.price_yen : NaN;

    return { ...it, qtyExpr: raw, qtyNum, lineTotal };
  });

  const total = lines.reduce(
    (sum, l) => sum + (Number.isFinite(l.lineTotal) ? l.lineTotal : 0),
    0
  );

  return { lines, total };
}

```

Also fix these endpoints:

```app.post("/cart/add", (req, res) => {
  const id = Number(req.body.itemId);
  const qty = parseQuantity(req.body.quantity);

  if (!Number.isFinite(id) || !Number.isFinite(qty)) {
    return res.redirect("/");
  }

  const cart = readCart(req);
  cart[id] = (cart[id] || 0) + qty;
  writeCart(res, cart);
  return res.redirect(`/challenge/#added-${id}`);
});
```

```
app.post("/cart/update", (req, res) => {
  const id = Number(req.body.itemId);
  const qty = parseQuantity(req.body.quantity);

  if (!Number.isFinite(id)) return res.redirect("/");

  const cart = readCart(req);

  if (!Number.isFinite(qty) || qty <= 0) {
    delete cart[id];
  } else {
    cart[id] = qty;
  }

  writeCart(res, cart);
  return res.redirect(`/challenge/#cart-item-${id}`);
});
```
