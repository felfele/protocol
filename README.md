# Felfele Social Protocol

In this article we will describe a social protocol on the Ethereum Swarm decentralized cloud service. Our goal is to make the protocol comprehensive and practical so that it covers all the basic functionality, although due to the richness of social applications it does not thrive to be complete. This is not (yet) a specification of a standard, but rather a result of research that hopefully inspires others to share similar ideas, enable cooperation and compatibility.

Our goals include:
- good usability
- pretty good privacy
- anonymity
- self-sovereignity
- sustainability

## Table of contents

- [Identities and Authentication](#Identities-and-Authentication)
- [Discovery](#Discovery)
    - [Public Names & Verifiable Identities](#Public-Names-%26-Verifiable-Identities)
    - [Handshake Protocol](#Handshake-Protocol)
    - [Key-exchange Protocol](#Key-exchange-Protocol)
- [Private Channel Protocol](#Private-Channel-Protocol)
- [Group Protocol](#Group-Protocol)
    - [Access Control](#Access-Control)
    - [Temporary Access](#Temporary-Access)
- [Posts](#Posts)
    - [Public Posts](#Public-Posts)
    - [Recent Post Feed](#Recent-Post-Feed)
    - [Private Posts](#Private-Posts)
- [Real-time Data](#Real-time-Data)
    - [Chat](#Chat)
    - [Streaming and Recording](#Streaming-and-Recording)
- [Multi-device Access](#Multi-device-Access)
    - [Authenticating with QR Code](#Authenticating-with-QR-Code)
    - [Web Login](#Web-Login)
- [Backup & Recovery](#Backup-%26-Recovery)
    - [Social Recovery](#Social-Recovery)
    - [Link Encryption](#Link-Encryption)
    - [Private Network](#Private-Network)


## Identities and Authentication

We are using [Ethereum key pairs](https://en.wikipedia.org/wiki/Ethereum#Addresses) for representing identities for people and for devices as well. Sometimes we are generating ephemeral key pairs for initiating an encrypted connection. The key pairs are relatively cheap to generate (~1msec on a mobile device) and they can also be generated offline.

When representing personally identifiable information such as a person in a social app or a mobile device it's generally good practice to keep the private key and the public key secret and only reveal the address publicly. As we will see later (see [Handshake](#Handshake)) this can provide enough information to do discovery and this allows the user to be more selective about revealing their identities.

By using Ethereum key pairs as the identities in the protocol means that they may also act as wallets and we can use this fact for integrating payments and incentivization models in our applications. However it seems that the best practice is to use different Ethereum identities as money-holding wallets than the identities in the social application because of privacy, anonymity and usability reasons (`TODO maybe elaborate?`)

There is also a well known issue about Ethereum key pairs that they are basically very long numbers that are impossible for people to memorize so we have to rely on our applications to keep them secret and only provide access to authorized users. This can be more easily done on mobile with encryption and biometric identification and it's a more challenging in the browser because at the moment there is no standardised API to do this. So the status quo is to rely on 3rd party installations in the form of browser extensions and it raises trust and usability questions.

We address some of this issues later (see [Verifiable Identities](#Verifiable-Identities), [Web Login](#Web-Login) and [Backup & Recovery](#Backup-%26-Recovery)) but we also work with the assumption that social protocols and applications are better suited for mobile platforms just because almost everyone has a mobile device handy and that's also where most of the content creation and consumption happens anyway.

Finally, authentication in this system relies on the knowledge of the private key and this can be used to calculate shared secrets or sign messages.

## Discovery

In social applications the primary interactions happen between different people so a big question is how those people find each other and make connections.

There are many ways how this can be achieved, but we can say that from an application perspective there are two main approaches exist: public and private. In public social apps there is usually a searchable database of all users where they can be searched by name or handle. In private apps there are no such thing and discovery usually relies on existing personal connections, often using previous generations of social apps as a way to initiate contact (for example chat apps using phone numbers from contact list).

It turns out that is possible to have a system where these two approaches can coexist. Users can decide if they want to be publicly discoverable, or even verifiable with their real name if they wish. At the same time other users may wish to remain invisible and maybe using different identities for different social circles and applications.

### Public Names & Verifiable Identities

`TODO blockchain & ENS`

### Handshake Protocol

For those people and applications that wish to remain anonymous and undiscoverable by default the protocol offers a way for two parties to establish an encrypted channel so that they can exchange information later privately. We call this the Handshake protocol and it is a 4-step process. The protocol usually starts with one party either scanning a QR code in person or sending a link to the other on an existing channel.

We would like this protocol have the following properties:
- The handshake produces a shared secret that can be used with a bulk encryption cypher for exchanging further messages
- A man-in-the-middle can be detected with the help of verification
- Past handshakes cannot be replayed

`TODO describe the protocol and verification with illustration`

There are two participants in the protocol, Alice and Bob. Both of them generate a random, ephemeral key pair, only used during the handshake (`HandshakeKeyPair`).

1. The first step of the protocol is that Alice also generates a random, ephemeral key pair (`SharedKeyPair`) and shares its private key with Bob, for example in the form of a QR code or a link. The shared private key is used to have a feed address that is known by both parties and they can also write in it with a convention defined in the protocol.

2. Then Alice waits for Bob to write a [hash commitment](https://en.wikipedia.org/wiki/Commitment_scheme)  as the 0th element of the feed. The hash commitment is the hash of Bob's public key of his `HandshakeKeyPair`. This is used to detect man-in-the-middle attacks with the protocol.

3. After this Alice shares her public key of her `HandshakeKeyPair` as the 1st element of the feed.

4. Finally Bob shares his public key of his `HandshakeKeyPair` as the 2nd element of the feed. Alice verifies that the hash of Bob's public key is indeed equal to what Bob sent in the 2nd step.

At this point they both know each others' public keys and they can calculate a shared secret with it. It's advised to do an extra verification step out of band to detect that there were no man-in-the-middle by calculating a [short hash](https://en.wikipedia.org/wiki/ZRTP#SAS) from the shared secret and displaying it to the users.

The communication after this happens at two addresses calculated with a convention. Alice's and Bob's updates will be found in their respective address that of their `HandshakeKeyPair` with the topic set to the hash of the shared secret.

Once there is an established, ephemeral, encrypted channel the parties can decide how they want to use it. It's possible that they just want to send one-time information to each other without revealing their real identities. Or they can decide to use this channel to [exchange their public key](#Key-exchange-Protocol) belonging their identities in a social app, therefore they can stay in touch and send messages later too.

### Key-exchange Protocol

After having an encrypted channel, public key exchange is really just publishing one's public keys to others. Once two parties know each others' key, a new secret can be calculated and that can be the basis of a [private channel communication](#Private-Channel-Protocol), preferably as a seed to something have more advanced secrecy properties, for example a triple-ratchet protocol.

This applies to [group messaging](#Group-Protocol) as well.

## Private Channel Protocol

In many cases it is desirable that the communication channel is set up between exactly two parties. In theory this could be achieved as a special case for a [group messaging protocol](#Group-Protocol) but in practice we found that it's better to have a separate protocol for this use-case.

Usually this means that an application maintains a list of identities whom with it already did a public key exchange (e.g. contacts) and actively looks for updates at a known location that is defined with a convention.

In our case the `address` is the address belonging to the public key of the peer and `topic` is defined to be the Keccak hash of the shared secret between the keypairs.

`TODO describe messages`

## Group Protocol

It is very common in social applications that more than two people are involved in the communication. The obvious solution would be to give access to some kind of shared resource, but that would practically require to share one or more private keys, which has many disadvantages. Instead we can implement this in a decentralized system by aggregating the individual updates and displaying them as if the group were updated.

In the protocol a group is created by one person who later can invite other people. Everyone else can be invited by either using an existing [private channel](#Private-Channel-Protocol) or creating an ad-hoc one-time channel with the help of the [Handshake protocol](#Handshake-Protocol).

`TODO describe invite protocol, tradeoffs`

### Access Control

`TODO describe CRDT`

### Temporary Access

`TODO describe temporary access`

## Posts

Until now we talked about how we can model the different types of social interactions which are very common in everyday situation, but we haven't described how to send the actual content with these protocols.

`TODO describe Post model, how it is stored on bzz`

### Public Posts

It is also possible sharing posts publicly by not encrypting them. The posts are stored in a [timeline](https://erebos.js.org/docs/timeline-spec), so once we have a reference to a post we can find all the previous ones as well.

### Recent Post Feed

Most social app displays the most recent posts in a news feed and in order to provide better user experience we can also store the last `N` updates (or a shortened version of them) as one update at a known location defined by convention. Then the application only has to do one feed lookup which returns a single hash (that can be cached) and with one more `bzz` lookup it can fetch the latest updates if necessary. This is very similar how [RSS](https://en.wikipedia.org/wiki/RSS) works and indeed it was modeled based on that.

```
interface RecentPostFeed {
    name: string;
    url: string;
    feedUrl: string;
    favicon: string;
    posts: PublicPost[];
    authorImage: ImageData;
    publicKey?: string;
}
```

### Private Posts

`TODO describe how content is encrypted, selective sharing`


## Real-time Data
### Chat
### Streaming and Recording

## Multi-device Access
### Authenticating with QR Code
### Web Login

## Backup & Recovery
### Social Recovery
### Link Encryption
### Private Network
