# Felfele Social Protocol

In this article we will describe a social protocol on the Ethereum Swarm decentralized cloud service. Our goal is to make the protocol comprehensive and practical so that it covers all the basic functionality, although due to the richness of social applications it does not thrive to be complete. This is not (yet) a specification of a standard, but rather a result of research that hopefully inspires others to share similar ideas, enable cooperation and compatibility.

Our goals include:
- good usability
- pretty good privacy
- anonymity
- self-sovereignity
- sustainability

## Table of contents

- [Identities and Authentication](#Identities%20and%20Authentication)
- [Discovery](#Discovery)
    - [Public Names & Verifiable Identities](#Public%20Names%20%26%20Verifiable%20Identities)
    - [Handshake Protocol](#Handshake%20Protocol)
    - [Key-exchange Protocol](#Key-exchange%20Protocol)
- [Private Channel Protocol](#Private%20Channel%20Protocol)
- [Group Protocol](#Group%20Protocol)
    - [Access Control](#Access%20Control)
    - [Temporary Access](#Temporary%20Access)
- [Posts](#Posts)
    - [Public Posts](#Public%20Posts)
    - [Recent Post Feed](#Recent%20Post%20Feed)
    - [Private Posts](#Private%20Posts)
- [Real-time Data](#Real-time%20Data)
    - [Chat](#Chat)
    - [Streaming and Recording](#Streaming%20and%20Recording)
- [Multi-device Access](#Multi-device%20Access)
    - [Authenticating with QR Code](#Authenticating%20with%20QR%20Code)
    - [Web Login](#Web%20Login)
- [Backup & Recovery](#Backup%20%26%20Recovery)
    - [Social Recovery](#Social%20Recovery)
    - [Link Encryption](#Link%20Encryption)
    - [Private Network](#Private%20Network)


## Identities and Authentication

We are using [Ethereum key pairs](https://en.wikipedia.org/wiki/Ethereum#Addresses) for representing identities for people and for devices as well. Sometimes we are generating ephemeral key pairs for initiating an encrypted connection. The key pairs are relatively cheap to generate (~1msec on a mobile device) and they can also be generated offline.

When representing personally identifiable information such as a person in a social app or a mobile device it's generally good practice to keep the private key and the public key secret and only reveal the address publicly. As we will see later (see [Handshake](#Handshake)) this can provide enough information to do discovery and this allows the user to be more selective about revealing their identities.

By using Ethereum key pairs as the identities in the protocol means that they may also act as wallets and we can use this fact for integrating payments and incentivization models in our applications. However it seems that the best practice is to use different Ethereum identities as money-holding wallets than the identities in the social application because of privacy, anonymity and usability reasons (`TODO maybe elaborate?`)

There is also a well known issue about Ethereum key pairs that they are basically very long numbers that are impossible for people to memorize so we have to rely on our applications to keep them secret and only provide access to authorized users. This can be more easily done on mobile with encryption and biometric identification and it's a more challenging in the browser because at the moment there is no standardised API to do this. So the status quo is to rely on 3rd party installations in the form of browser extensions and it raises trust and usability questions.

We address some of this issues later (see [Verifiable Identities](#Verifiable%20Identities), [Web Login](#Web%20Login) and [Backup & Recovery](#Backup%20%26%20Recovery)) but we also work with the assumption that social protocols and applications are better suited for mobile platforms just because almost everyone has a mobile device handy and that's also where most of the content creation and consumption happens anyway.

Finally, authentication in this system relies on the knowledge of the private key and this can be used to calculate shared secrets or sign messages.

## Discovery

In social applications the primary interactions happen between different people so a big question is how those people find each other and make connections.

There are many ways how this can be achieved, but we can say that from an application perspective there are two main approaches exist: public and private. In public social apps there is usually a searchable database of all users where they can be searched by name or handle. In private apps there are no such thing and discovery usually relies on existing personal connections, often using previous generations of social apps as a way to initiate contact (for example chat apps using phone numbers from contact list).

It turns out that is possible to have a system where these two approaches can coexist. Users can decide if they want to be publicly discoverable, or even verifiable with their real name if they wish. At the same time other users may wish to remain invisible and maybe using different identities for different social circles and applications.

### Public Names & Verifiable Identities

`TODO blockchain & ENS`

### Handshake Protocol

For those people and applications that wish to remain anonymous and undiscoverable by default the protocol offers a way for two parties to establish an encrypted channel so that they can exchange information later privately. We call this the Handshake protocol and it is a 4-step process. The protocol usually starts with one party either scanning a QR code in person or sending a link to the other on an existing channel.

`TODO describe the protocol and verification with illustration`

Security properties of the handshake protocol:
- The handshake produces a shared secret that can be used with a bulk encryption cypher for exchanging further messages.
- A man-in-the-middle can be detected with the help of verification
- Past handshakes cannot be replayed.

Once there is an established, ephemeral, encrypted channel the parties can decide how they want to use it. It's possible that they just want to send one-time information to each other without revealing their real identities. Or they can decide to use this channel to [exchange their public key](#Key-exchange%20Protocol) belonging their identities in a social app, therefore they can stay in touch and send messages later too.

### Key-exchange Protocol

After having an encrypted channel, public key exchange is really just publishing one's public keys to others. Once two parties know each others' key, a new secret can be calculated and that can be the basis of a [private channel communication](#Private%20Channel%20Protocol), preferably as a seed to something have more advanced secrecy properties, for example a triple-ratchet protocol.

This applies to [group messaging](#Group%20Protocol) as well.

## Private Channel Protocol

In many cases it is desirable that the communication channel is set up between exactly two parties. In theory this could be achieved as a special case for a [group messaging protocol](#Group%20Protocol) but in practice we found that it's better to have a separate protocol for this use-case.

Usually this means that an application maintains a list of identities whom with it already did a public key exchange (e.g. contacts) and actively looks for updates at a known location that is defined with a convention.

In our case the `address` is the address belonging to the public key of the peer and `topic` is defined to be the Keccak hash of the shared secret between the keypairs.

`TODO describe messages`

## Group Protocol

It is very common in social applications that more than two people are involved in the communication. The obvious solution would be to give access to some kind of shared resource, but that would practically require to share one or more private keys, which has many disadvantages. Instead we can implement this in a decentralized system by aggregating the individual updates and displaying them as if the group were updated.

In the protocol a group is created by one person who later can invite other people. Everyone else can be invited by either using an existing [private channel](#Private%20Channel%20Protocol) or creating an ad-hoc one-time channel with the help of the [Handshake protocol](#Handshake%20Protocol).

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
