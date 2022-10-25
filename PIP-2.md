# PIP-2 Point Notifications

## Status

WIP, very rough draft

Pull request: https://github.com/pointnetwork/point-improvement-proposals/pull/3

## Motivation

The current way of emitting and querying notification events in Ethereum Virtual Machine is to use _logs_. Using logs is cheaper than normal storage operations, but querying for such events is cumbersome, it requires indexing of the whole chain on the node side, and often is slow and rate-limited or block-range-limited.

Especially in view of our plans of all users running their own light node for trust minimization, relying on indexing the whole chain would become a large hindrance to it.

In this proposal we sacrifice gas efficiency for ease of use and performance by storing notifications in the traditional EVM storage, with automatic indexes done by EVM and incorporated into the block states.

We also add a helper contract signaling to Point Engine which notifications the current user is subscribed to, so that notification settings can be restored on any device without any extra settings storage.

## Idea

We already do this in some of our dApps to not rely on emitting events and querying logs.

In Point Social, instead of relying on the Engine scanning the whole blockchain for events, we auto-increment IDs of posts globally, posts within each user profile, and comments within a post, and store them sequentially in arrays, which makes it easier to query by simply iterating over the array until we hit `0x0` (empty row).

In Identities, instead of relying on the Engine scanning the whole blockchain for new identities and subidentities events, we store them sequentially in an array.

This proposal generalizes such approach to create one standard to proceed with and allow other dApps to benefit from it too.

The following is a step-by-step walkthrough explaining the reasoning and using examples to make it easier to implement and improve upon.

## Notifications Contract

We would deploy a contract global to the Point Chain and call it `PointNotificationsContract`.

```solidity
struct PointNotification {
    // TODO: propose a more efficient and flexible version in case some of the arguments are not needed and we're wasting space
    address app,
    address sender,
    bytes32 topic0,
    bytes32 topic1,
    bytes32 topic2,
    bytes32 data0,
    bytes32 data1,
    uint datetime
}

mapping(address => PointNotification[]) public notifications;

function notify(bytes32 recipient, PointNotification memory notification) {
    notification.app = msg.sender; // Notice that msg.sender is the contract that is interacting with this contract
    notifications[recipient].push(notification);
}
```

Suppose we want Point Mail users to be able to quickly identify and iterate over new emails. Then, `PointMail` contract would call this `notify()` function of `PointNotificationsContract`.

`sender` represents the email sender, which `PointMail` contract should pass (although we shouldn't rely on it for security on the client side, because it can be spoofed by the app contract)

## Subscribing and unsubscribing

In the implementation above, we don't have any measures against dApps spamming all kinds of users with all kinds of unwanted notifications. To resolve this, we add the functionality of subscribing and unsubscribing to notifications.

```solidity
// In this struct, fields are not holding data, instead they're holding filters
// If any field is 0x0, it means no filter on this field. Any value other than 0x0 means the exact value has to be matched in order for it to be shown to the user.
struct PointNotificationSubscription {
    address app,
    address sender,
    bytes32 topic0,
    bool is_group // will become obvious later
}

// the first address is the notification recipient
// the second address is the app (contract address)
// the bytes32 is topic0
mapping(address => address => bytes32 => bool[]) public active_subscriptions;

mapping(address => PointNotificationSubscription[]) public subscriptions;

function subscribe(address app, bytes32 sender, bytes32 topic0) { 
    subscriptions[ msg.sender ].push(PointNotificationSubscription( // in this case, msg.sender is the user calling this function
        app
        sender,
        topic0,
        false // is_group
    ));
    active_subscriptions[ msg.sender ][ app ][ topic0 ] = true; // in this case, msg.sender is the user calling this function
}

function unsubscribe(address app, bytes32 topic0) {
    active_subscriptions[ msg.sender ][ app ][ topic0 ] = false;
}

function notify(bytes32 recipient, PointNotification memory notification) {
    // TODO: before we notify the user, we would make sure that the user has subscribed to the notification using subscriptions mapping
    // Important: if the subscription has not been found, don't revert the whole execution or throw errors, simply ignore it so that the caller contract can proceed
}
```

## Subscribing to notifications

A dApp frontend, such as Point Social, would at some point ask the user to subscribe to the notifications.

To do that, the dApp frontend would initiate a new direct transaction to `PointNotificationContract`'s `subscribe` method. Confirmation window would recognize that it's a special type of transaction, and would apply special formatting rules for the user, explaining what the user is subscribing to.

A UI in Point Home would allow to see subscriptions and unsubscribe if needed.

## Groups

There are situations, where the notification's reach is not one-to-one or one-to-few, like in Point Mail, but one-to-many, like in Point Social. For example, if 1,000 people subscribed to one person, when that person adds a new post, sending notifications to all 1,000 people would be infeasible.

Instead, we introduce the notion of notification _groups_, and we represent them as virtual users, whose address is a hash of the notification signature (partial, so we can allow for filters by using several groups). Real users can subscribe to these groups. Then, an application would send only one notification to this virtual group.

```solidity

mapping(address => address => bytes32 => bool[]) public subscriptions;

// leave any field as 0x0 to not have any filter on this field
function subscribeToGroup(address app, bytes32 recipient, address sender, topic0) {
    group = keccak256(app, sender, topic0);
    subscriptions[ msg.sender ].push(PointNotificationSubscription(
        group, // instead of app
        0x0,
        0x0,
        true // is_group
    ));
    active_subscriptions[ msg.sender ][ group ][ 0x0 ] = true; // in this case, msg.sender is the user calling this function
}

function notify(bytes32 recipient, PointNotification memory notification) {
    notification.app = msg.sender;

    if (recipient == 0x0) {
        // if the recipient is 0x0, it's a group, create virtual users
        recipient1 = keccak256(notification.app, 0x0, 0x0)
        recipient2 = keccak256(notification.app, 0x0, notification.topic0)
        recipient3 = keccak256(notification.app, notification.sender, 0x0)
        recipient4 = keccak256(notification.app, notification.sender, notification.topic0)

        // Since it's unlikely that a hash would coincide with any user's address, we don't check the permissions
        
        // ...add the notifications to each of the virtual users
    } else {
        // 
        
        // ...add the notification
    }    
}
```

## Querying for notifications on the client side

At this point, for each user, we have an array of subscriptions which holds both subscriptions to apps and to groups, and a map tree to check whether the subscription is still active.

Point Engine, when online, would update this list, which is facilitated by it being an iterable array.

Then, based on whether `is_group` is set to false or true, it would either scan for the new notifications for its user, or scan the notifications for the group's "virtual user".

The engine keeps track of which notifications have already been shown, and only starts iterating from that point onwards, until it hits an empty row.

## Displaying notifications

For now, we don't need to do push notifications or even OS notifications, simply having a UI to see if there's any new unread notifications for the user would make this feature PoC-ready.

The notifications are sourced from several arrays, so when combined, they should be sorted by the datetime field.

## Unread notifications on other devices

The engine keeps track of read and unread notifications in its database. If a user then changes devices, the engine will not have this information, and should assume all notifications up until this point to be considered as read.

## Other questions

### Notifications contract ownership

For now (especially in the development phase), we can have the contract upgradeable like we do with other contracts, and then gradually move to switching the ownership to a DAO etc., but even if the contract is not upgradeable and something breaks, we would be able to create another contract and point clients to it instead, migrating the storage. Upgrading is not of such a high concern because the contract doesn't store any funds or perform any significant security operations.
