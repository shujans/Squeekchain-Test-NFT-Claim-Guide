# 🐭 Squeekchain Test NFT Claim Guide

Dhakacrypto Community Guide

Squeekchain-এর Sepolia Test NFT Mint পেজ চালু হয়েছে। তবে ওয়েবসাইটের নিজস্ব RPC সমস্যা থাকায় অনেকের Mint বাটন ঠিকভাবে কাজ করছে না।

এই সমস্যার বিকল্প হিসেবে একটি Browser Console Script শেয়ার করা হয়েছে, যার মাধ্যমে নিজের Community Tier অনুযায়ী বরাদ্দকৃত Test NFT Claim করা সম্ভব।

🔗 Official Test Mint Page:  
https://sepolia.squeekchain.com/mint

📌 গুরুত্বপূর্ণ তথ্য

- এটি শুধুমাত্র Test NFT Distribution।
- এটি Official 9,999 Genesis NFT Sale নয়।
- NFT Claim করার জন্য ETH মূল্য দিতে হবে না।
- তবে Sepolia Network-এর Gas Fee প্রয়োজন হবে।

## 🎁 NFT Allocation

| Community Tier | NFT Allowance |
|---|---:|
| 🥇 Gold | 6 |
| 🥈 Silver | 4 |
| 🥉 Bronze | 1 |
| Member | 1 |
| Unranked | 1 |

## 🎥 Video Tutorial

Claim করার সম্পূর্ণ প্রক্রিয়াটি ভিডিওতে দেখতে পারো:

https://x.com/dhakacrypto_/status/2118352446800163021

## ⚙️ কী কী প্রয়োজন হবে?

Claim করার আগে নিচের বিষয়গুলো নিশ্চিত করো:

1. Rabby অথবা MetaMask Wallet।
2. Genesis-এ লিংক করা একই Wallet Address।
3. Sepolia Testnet ETH (Gas Fee-এর জন্য)।
4. Official Squeekchain Test Mint পেজে অ্যাক্সেস।

Sepolia ETH Faucet:  
https://faucet.quicknode.com/ethereum/sepolia

## 📝 NFT Claim করার নিয়ম

Step 1 — Mint Website ওপেন করো

প্রথমে নিচের লিংকে প্রবেশ করো:

https://sepolia.squeekchain.com/mint

স্ক্রিপ্টটি ব্যবহার করার সময় এই পেজেই থাকতে হবে। কারণ Website Session এবং CORS Request এই পেজের সঙ্গে সম্পর্কিত।

Step 2 — Wallet Connect করো

Rabby অথবা MetaMask Wallet থেকে Sepolia Network সিলেক্ট করো।

Genesis-এ যে Wallet Address লিংক করেছিলে, সেটিই ব্যবহার করবে।

Wallet-এ Gas Fee দেওয়ার মতো পর্যাপ্ত Sepolia ETH থাকতে হবে।

Step 3 — Browser Console ওপেন করো

- কিবোর্ড থেকে `F12` চাপো।
- Developer Tools থেকে `Console` ট্যাব ওপেন করো।
- ব্রাউজার Paste করার অনুমতি চাইলে `allow pasting` লিখে Enter দাও।
- নিচে দেওয়া JavaScript Script কপি করে Console-এ Paste করো।
- এরপর Enter চাপো।

Step 4 — Wallet Signature ও Transaction Confirm করো

আগে থেকে Sign In করা না থাকলে Wallet একটি Login Message Sign করার অনুরোধ করতে পারে।

এরপর NFT Claim Transaction Confirm করতে বলবে।

Confirm করার আগে অবশ্যই তিনটি বিষয় যাচাই করবে:

1. Network যেন Sepolia হয়।
2. Transaction Value যেন 0 ETH হয়।
3. Destination Contract যেন Console-এ দেখানো `authMinter` Address-এর সঙ্গে মিলে।

Step 5 — Transaction Verify করো

Transaction সফলভাবে Submit হলে Console-এ একটি Transaction Hash দেখা যাবে।

সেটি নিচের Explorer-এ গিয়ে চেক করতে পারবে:

https://sepolia.etherscan.io

## 📋 Original Claim Script
```javascript
// Squeekchain Sepolia test NFT claim
// Run this in the browser console on https://sepolia.squeekchain.com/mint
// Uses the official community API, then sends claim() through your injected wallet (Rabby / MetaMask).
// Cost is 0 ETH plus Sepolia gas. This is the test distribution, not the production Genesis sale.

(async () => {
  const API = 'https://community.squeekchain.com/v1/community';
  const SEPOLIA = 11155111;
  if (!window.ethereum) throw new Error('No injected wallet found');
  try {
    await ethereum.request({ method: 'wallet_switchEthereumChain', params: [{ chainId: '0xaa36a7' }] });
  } catch (e) {
    if (e.code !== 4902) throw e;
    await ethereum.request({
      method: 'wallet_addEthereumChain',
      params: [{
        chainId: '0xaa36a7',
        chainName: 'Sepolia',
        nativeCurrency: { name: 'Sepolia ETH', symbol: 'ETH', decimals: 18 },
        rpcUrls: ['https://ethereum-sepolia-rpc.publicnode.com'],
        blockExplorerUrls: ['https://sepolia.etherscan.io']
      }]
    });
  }
  const [me] = await ethereum.request({ method: 'eth_requestAccounts' });
  const addr = me.toLowerCase();
  const api = async (path, { method = 'GET', token, body } = {}) => {
    const headers = { 'content-type': 'application/json' };
    if (token) headers.authorization = 'Bearer ' + token;
    const res = await fetch(API + path, {
      method,
      headers,
      body: body !== undefined ? JSON.stringify(body) : undefined,
      credentials: 'omit',
      cache: 'no-store'
    });
    const text = await res.text();
    let data = null;
    try { data = text ? JSON.parse(text) : null; } catch {}
    if (!res.ok) throw new Error((data && data.error) || String(res.status));
    return data;
  };
  let session = null;
  try { session = JSON.parse(localStorage.getItem('squeek.community.session') || 'null'); } catch {}
  if (session && Date.parse(session.expiresAt) <= Date.now()) session = null;
  if (!session || !session.token) {
    const ch = await api('/auth/challenge', { method: 'POST', body: { address: addr, chainId: SEPOLIA } });
    const signature = await ethereum.request({ method: 'personal_sign', params: [ch.message, addr] });
    session = await api('/auth/verify', { method: 'POST', body: { nonce: ch.nonce, signature } });
    localStorage.setItem('squeek.community.session', JSON.stringify({ token: session.token, expiresAt: session.expiresAt }));
    localStorage.setItem('squeek.community.signedInBefore', '1');
  }
  const status = await api('/mint/status', { token: session.token });
  console.log('mint status', status);
  if (!status.claimsOpen) throw new Error('claims are not open');
  if (!status.recipient) throw new Error('no eligible recipient, link a sepolia wallet in genesis');
  if (status.recipient.toLowerCase() !== addr) throw new Error('wrong wallet, connect ' + status.recipient);
  if (!status.available) throw new Error('nothing left to claim, already ' + status.claimed + '/' + status.allowance);
  const fresh = (a) =>
    a &&
    a.chainId === SEPOLIA &&
    a.recipient.toLowerCase() === addr &&
    BigInt(a.deadline) > BigInt(Math.floor(Date.now() / 1000)) &&
    Date.parse(a.expiresAt) > Date.now();
  const auth = fresh(status.authorization)
    ? status.authorization
    : await api('/mint/authorization', { method: 'POST', token: session.token, body: {} });
  console.log('authorization', auth);
  if (auth.recipient.toLowerCase() !== addr) throw new Error('authorization recipient mismatch');
  const pad = (h) => h.replace(/^0x/, '').toLowerCase().padStart(64, '0');
  const sig = auth.signature.replace(/^0x/, '').toLowerCase();
  const data =
    '0x3374b3df' +
    pad(auth.recipient) +
    pad(BigInt(auth.quantity).toString(16)) +
    pad(auth.authorizationId) +
    pad(BigInt(auth.deadline).toString(16)) +
    pad('a0') +
    pad((sig.length / 2).toString(16)) +
    sig.padEnd(Math.ceil(sig.length / 64) * 64, '0');
  console.log('etherscan write contract', {
    to: auth.authMinter,
    recipient: auth.recipient,
    quantity: auth.quantity,
    authorizationId: auth.authorizationId,
    deadline: auth.deadline,
    signature: auth.signature
  });
  const hash = await ethereum.request({
    method: 'eth_sendTransaction',
    params: [{ from: me, to: auth.authMinter, value: '0x0', data }]
  });
  console.log('claim tx', hash);
  return hash;
})(); ```


## 🔍 Script কীভাবে কাজ করে?

স্ক্রিপ্টটি মূলত তিনটি কাজ করে:

1. Mint Website-এ থাকা Genesis Session ব্যবহার করে। Session না থাকলে Squeekchain-এর Official Community API-এর মাধ্যমে Sign In করে।
2. API থেকে `/v1/community/mint/status` এবং `/v1/community/mint/authorization` ব্যবহার করে Claim Status ও Authorization সংগ্রহ করে।
3. Wallet-এর মাধ্যমে Official Auth Minter Contract-এ NFT Claim Transaction পাঠায়।

## ⚠️ Common Errors & Solutions

1. `signup_required` অথবা `human_verification_failed`

এই Error দেখা গেলে প্রথমে Genesis Registration সম্পন্ন করতে হবে।

Genesis Website:  
https://squeekchain.com/genesis

2. `wrong wallet`

এই Error এলে বুঝতে হবে Connected Wallet এবং Genesis-এ লিংক করা Wallet এক নয়।

Console-এ দেখানো Recipient Address-এর সঙ্গে নিজের Connected Wallet মিলিয়ে নাও।

## 🔐 Security Guidelines

- কখনোই নিজের Seed Phrase বা Private Key কারও সঙ্গে শেয়ার করবে না।
- অপরিচিত Script সরাসরি Browser Console-এ চালাবে না।
- Wallet Signature এবং Transaction Details ভালোভাবে যাচাই করবে।
- শুধু Official Website ও Verified Source ব্যবহার করবে।
- কোনো সন্দেহজনক Transaction দেখলে Confirm করবে না।

Community Notice: এই গাইডটি কমিউনিটির শিক্ষামূলক তথ্যের জন্য তৈরি। মূল Script ও Claim Method-এর কৃতিত্ব মূল Developer-এর। এটি কোনো Official Squeekchain Announcement নয়।

Stay Updated | Verify Everything | Stay Safe

— Dhakacrypto Community
