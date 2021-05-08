# The Director
 The Director is a special research project of Riff.CC, intended to completely decentralise the primary mechanisms that
 allow Riff.CC to operate.

## Purpose
 The Director creates a JSON-based "pseudo-API" suitable for consumption by applications, both SPA and traditional.
 It is intended to create a package of metadata which can be stored entirely on IPFS and loaded as needed.
 Once the package is created, The Director uses a special custom SLP token to permanently record the root IPFS hash.
 Optionally, The Director can also sign the entire bundle to certify it, by simply signing the root IPFS hash.

## Methodology
 In order to provide a verifiable chain of trust, we propose a surprisingly simple solution.

 1. Using the SLP standard, we create a custom for-purpose SLP token which can be used to provably certify metadata.
 2. Two new wallets are created.
 3. The SLP minting baton is used to mint tokens and send them to Wallet A.
 4. Wallet A sends tokens to Wallet B, attaching the root IPFS hash as an OP_RETURN code.

 When the Riff.CC application starts, it simply checks Wallet B's transactions, finds the latest valid one
 and uses it to load in the root hash.

 To handle future updates, we simply publish a new IPFS hash by sending SLP tokens from Wallet A to Wallet B.

 This system allows us to create a special key pair that can be used to certify and publish content to Riff.CC,
 while retaining the ability to switch to a new keypair in the process. As all key pairs must have tokens generated
 by the minting baton in order to be considered valid, any key pair with tokens must have been created by Riff.CC.

 It follows that any metadata and any IPFS hashes described the "pseudo-API" were verifiably certified by our keys,
 providing a level of end-to-end assurance that they are legitimate.

 A compromised key pair could allow an attacker to publish content and force Riff.CC users to see it.
 However, it's possible to permanently invalidate a compromised key pair - we simply issue a new key pair, then update
 the Origin Root pointer to include the new Wallet B address instead of the old one.

 Upon starting, the Riff.CC application will fetch the Origin Root's JSON data, and check the SLP addresses
 it returns. If the suggested address' SLP tokens were received by its associated Wallet A at a later point in time
 than Wallet A in the old address received theirs, the new key pair ("metadataSigner") is more trustworthy than the
 currently stored address, and the currently stored address is discarded in favour of the new one.

 If the suggested address' SLP tokens are older than those of the currently selected key pair, we know that either a
 malicious attacker has altered the contents of the Origin Root gateway to return an old and compromised key pair,
 or - much more likely - the CDN is storing old or cached data. We simply ignore it and continue using the newer pair.

 As alluded to earlier, digital signatures form the final piece of our proposed solution - as CDNs could potentially
 alter the contents of the JSON blobs they deliver, we need a way of mitigating that. We deal with it by signing the
 entire metadata bundle, which is in fact as simple as signing the root IPFS hash with the metadataSigner keys. Clients
 can check that the hash has a valid associated signature and throw a warning or failure if it appears to be wrong. This
 allows us to take advantage of CDNs to accelerate the delivery of the metadata bundles, without opening ourselves up to
 the possibility of malicious data being injected. We can additionally use the Cloudflare-developed IPFS security
 attributes to provide further assurances on the platforms which can support them.

## Ramifications
 In total this results in a system that allows for the issuing of new key pairs that can be distributed to less trusted
 parties due to the ease with which they can be revoked and made ineffective for attackers. The only known circumstance
 so far that allows for major attacks is if the Origin Root gateways are all down while a key is compromised,
 preventing us from making a new key pair take effect. Even in this circumstance, however, we could send a special
 "burn" transaction to the old Wallet B, with an OP_RETURN indicating that keypair has been burnt, followed by the
 next appropriate key pair to use. Seeing this, a client will switch to the new key pair without having to contact the
 Origin Root gateways.

 As this system helps facilitate a full and permanent switch to IPFS and the retirement of the BitTorrent protocol
 within the platform, it has a further advantage - as IPFS swarms are globally shared by default, any content we seed
 is provided to the rest of the IPFS ecosystem with no work required by us. This does potentially allow room for a
 competitor project to overtake and out-compete us, but in this event we have either failed in our mission or they are
 bringing interesting new ideas and passion to the space. In any case, most competitors will simply live alongside us
 and help boost the overall goals of both projects.
 
 If this research pans out, clients will be able to load content from Riff.CC with zero infrastructure of our own being
 required, as all contents can be loaded from IPFS or an IPFS gateway and the pointers for where to retrieve that data
 from are stored publicly on the BCH chain. We retain the ability to update the metadata and remove or edit content, to
 improve it or to remove objectionable or illegal materials. Any content references in the old metadata remains on the
 IPFS network, but is no longer likely to be pinned by any Riff.CC users and no longer earns any BONS, reducing any
 incentive for our network of users to continue seeding it.

 The implementation of such a system with open and transparent code would allow any platform to create their own "view" of the world, consuming Riff.CC's content and user bandwidth in a symbiotic fashion, or some other set of content or some combination of both.