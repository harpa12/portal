# Rust Canister Development Security Best Practices

## Immutable Smart Contracts

### Use a decentralized governance system like SNS to make a canister have a decentralized controller

#### Security Concern

The controller of a canister can change / update the canister whenever they like. If a canister e.g. stores assets such as ICP, this effectively means that the controller can steal these by updating the canister and transfer the cycles to their account.

#### Recommendation

-   To implement an immutable canister, you cannot be the controller of the canister. Control must be passed to the service nervous system (SNS) or a decentralized governance system.

-   Another option to create an immutable canister is to remove the canister controller completely. But note that this implies that the canister cannot be upgraded, which may have severe implications in case e.g. a bug would be found.

-   Note that, contrary to some other blockchains, also immutable smart contracts need cycles to run, and they can receive cycles.

### Verify the immutability of smart contracts you depend on

#### Security Concern

If a canister depends on a canister (i.e. makes inter-canister calls to it), it is essential that the canister is immutable. Otherwise, i.e. if it has a controller, they could modify the smart contract whenever they like, e.g. to steal assets held by the canister.

#### Recommendation

If you interact with a canister that must be an immutable smart contract, make sure it is controlled by the NNS, service nervous system (SNS) or a decentralized governance system.

## Authentication

### Make sure any action that only a specific user should be able to do requires authentication

#### Security Concern

If this is not the case, an attacker may be able to perform sensitive actions on behalf of a user, compromising their account.

#### Recommendation

-   By design, for every canister call the caller can be identified. The calling [principal](../ic-interface-spec.md#principals) can be accessed using the system API’s methods `ic0.msg_caller_size` and `ic0.msg_caller_copy` (see [here](../ic-interface-spec.md#system-api-imports)). If e.g. Internet Identity is used, the principal is the user identity for this specific origin, see [here](../ii-spec.md#identity-design-and-data-model). If some actions (e.g. access to user’s account data or account specific operations) should be restricted to a principal or a set of principals, then this must be explicitly checked in the canister call, for example as follows in Rust:

<!-- -->

        // Let pk be the public key of a principal that is allowed to perform
        // this operation. This pk could be stored in the canister's state.
        if caller() != Principal::self_authenticating(pk) {  ic_cdk::trap(...) }

        // Alternatively, if the canister keeps data for different principals
        // in e.g. a map such as BTreeMap<Principal, UserData>, then the canister
        // must ensure that each caller can only access and perform operations
        // on their own data:
        if let Some(user_data) = user_data_store.get_mut(&caller()) {
            // perform operations on the user's data
        }

-   In Rust, the `ic_cdk` crate can be used to authenticate the caller using `ic_cdk::api::caller`. Make sure the returned principal is of type `Principal::self_authenticating` and identify the user’s account using the public key of that principal, see the example code above.

-   Do authentication as early as possible in the call to avoid unauthenticated actions and potentially expensive operations before authentication. It is also a good idea to [deny service to anonymous users](#disallow-the-anonymous-principal-in-authenticated-calls).

### Disallow the anonymous principal in authenticated calls

#### Security Concern

`ic0::api::caller` may also return `Principal::anonymous()`. In authenticated calls, this is probably undesired (and could have security implications) since this would behave like a shared account for anyone that does unauthenticated calls.

#### Recommendation

In authenticated calls, make sure the caller is not anonymous and return an error or trap if it is. This could e.g. be done centrally by using a helper method such as:

    fn caller() -> Result<Principal, String> {
        let caller = ic0::api::caller();
        // The anonymous principal is not allowed to interact with canister.
        if caller == Principal::anonymous() {
            Err(String::from(
                "Anonymous principal not allowed to make calls.",
            ))
        } else {
            Ok(caller)
        }
    }

## Asset Certification

### Use HTTP asset certification and avoid serving your dApp through `raw.ic0.app`

#### Security Concern

dapps on the IC can use [asset certification](https://wiki.internetcomputer.org/wiki/HTTP_asset_certification) to make sure the HTTP assets delivered to the browser are authentic (i.e. threshold-signed by the subnet). If an app does not do asset certification, it can only be served insecurely through `raw.ic0.app` , where no asset certification is checked. This is insecure since a single malicious node or boundary node can freely modify the assets delivered to the browser.

If an app is served through `raw.ic0.app` in addition to `ic0.app`, an adversary may trick users (phishing) into using the insecure raw.ic0.app.

#### Recommendation

-   Only serve assets through `<canister-id>.ic0.app` where the service worker verifies asset certification. Do not serve through `<canister-id>.raw.ic0.app`.

-   Serve assets using the asset canister (which creates asset certification automatically), or add the `ic-certificate` header including the asset certification as e.g. done in the [NNS dApp](https://github.com/dfinity/nns-dapp) or [Internet Identity](https://github.com/dfinity/internet-identity).

-   Check in the canister’s `http_request` method if the request came through raw. If so, return an error and do not serve any assets.

## Canister Storage

### Use `thread_local!` with `Cell/RefCell` for state variables and put all your globals in one basket.

#### Security Concern

Canisters need global mutable state. In Rust, there are several ways to achieve this. However, some options can lead e.g. to memory corruption.

#### Recommendation

-   [Use `thread_local!` with `Cell/RefCell` for state variables.](https://mmapped.blog/posts/01-effective-rust-canisters.html#use-threadlocal) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html))

-   [Put all your globals in one basket.](https://mmapped.blog/posts/01-effective-rust-canisters.html#clear-state) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html))

### Limit the amount of data that can be stored in a canister per user

#### Security Concern

If a user is able to store a big amount of data on a canister, this may be abused to fill up the canister storage and make the canister unusable.

#### Recommendation

Limit the amount of data that can be stored in a canister per user. This limit has to be checked whenever data is stored for a user in an update call.

### Consider using stable memory, version it, test it

#### Security Concern

Canister memory is not persisted across upgrades. If data needs to be kept across upgrades, a natural thing to do is to serialize the canister memory in `pre_upgrade`, and deserialize it in `post_upgrade`. However, the available number of instructions for these methods is limited. If the memory grows too big, the canister can no longer be updated.

#### Recommendation

-   Stable memory is persisted across upgrades and can be used to address this issue.

-   [Consider using stable memory.](https://mmapped.blog/posts/01-effective-rust-canisters.html#stable-memory-main) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html)). See also the disadvantages discussed there.

-   [Version stable memory.](https://mmapped.blog/posts/01-effective-rust-canisters.html#version-stable-memory) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html))

-   [Test the upgrade hooks.](https://mmapped.blog/posts/01-effective-rust-canisters.html#test-upgrades) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html))

-   See also the section on upgrades in [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister) (though focused on Motoko)

-   Write tests for stable memory to avoid bugs.

-   Some libraries (mostly work in progress / partly unfinished) that people work on:

    -   <https://github.com/dfinity/stable-structures/>

        -   HashMap: <https://github.com/dfinity/stable-structures/pull/1> (currently not prod ready)

    -   <https://github.com/seniorjoinu/ic-stable-memory-allocator>

-   See [Current limitations of the Internet Computer](https://wiki.internetcomputer.org/wiki/Current_limitations_of_the_Internet_Computer), sections "Long running upgrades" and "\[de\]serialiser requiring additional wasm memory"

-   For example, [internet identity](https://github.com/dfinity/internet-identity) uses stable memory directly to store user data.

### Consider encrypting sensitive data on canisters

#### Security Concern

By default, canisters provide integrity but not confidentiality. Data stored on canisters can be read by nodes / replicas.

#### Recommendation

-   Consider end-to-end encrypting any private or personal data (e.g. user’s personal or private information) on canisters.

-   The example dApp [Encrypted Notes](https://github.com/dfinity/examples/tree/master/motoko/encrypted-notes-dapp) illustrates how end-to-end encryption can be done.

### Create backups

#### Security Concern

A canister could be rendered unusable so it could never be upgraded again e.g. due to the following reasons:

-   It has a faulty upgrade process (due to some bug from the dapp developer).

-   The state becomes inconsistent / corrupt because of a bug in the code that persists data.

#### Recommendation

-   Make sure methods used in upgrading are tested or the canister becomes immutable.

-   It may be useful to have a disaster recovery strategy that makes it possible to reinstall the canister.

-   See the "Backup and recovery" section in [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister)

## Inter-Canister Calls and Rollbacks

### Don’t panic after await and don’t lock shared resources across await boundaries

#### Security Concern

Panics and traps roll back the canister state. So any state change followed by a trap or panic is of concern. This is also an important concern when inter-canister calls are made. If a panic/trap occurs after an `await` to an inter-canister call, then the state is reverted to the snapshot before the inter-canister call callback invocation (and not before the entire call!).

This may e.g. lead to the following issues:

-   If state changes before an inter-canister call leave the state inconsistent and there is a panic after the inter-canister call, this results in inconsistent canister state.

-   In particular, if allocated resources (e.g. locks or memory) from before an inter-canister call are not released this can e.g. lead to a canister being locked forever.

-   Generally, there can be bugs when data is not persisted when the developer expected it to be.

#### Recommendation

-   [Don’t panic after `await`](https://mmapped.blog/posts/01-effective-rust-canisters.html#panic-await) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html))

-   [Don’t lock shared resources across await boundaries](https://mmapped.blog/posts/01-effective-rust-canisters.html#dont-lock) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html))

-   See also: "Inter-canister calls" section in [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister)

-   For context: [IC interface spec on message execution](../ic-interface-spec#message-execution)

### Be aware that state may change during inter-canister calls

#### Security Concern

Messages (but not entire calls) are processed atomically. This can lead to security issues, such as:

-   Time-of-check time-of-use: checking some condition on global state before an inter-canister call and wrongly assuming it to still hold when the call returned.

#### Recommendation

-   Be aware that state may change during an inter-canister call. Carefully review your code so that this kind of bugs do not occur.

-   See also: "Inter-canister calls" section in [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister)

### Only make inter-canister calls to trustworthy canisters

#### Security Concern

-   If inter-canister calls are made to potentially malicious canisters, this can lead to DoS issues or there could be issues related to candid decoding. Also, the data returned from a canister call could be assumed to be trustworthy when it is not.

-   If a canister is called with a callback, the receiver can stall indefinitely if the peer does not respond, resulting in DoS. A canister can no longer be upgraded if it is in that state. Recovery would involve reinstalling, wiping the state of the canister.

-   In summary, this can DoS a canister, consume an excessive amount of resources, or lead to logic bugs if the behavior of the canister depends on the inter-canister call response.

#### Recommendation

-   Only make inter-canister calls to trustworthy canisters.

-   Sanitize data returned from inter-canister calls.

-   See "Talking to malicious canisters" section in [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister)

-   See [Current limitations of the Internet Computer](https://wiki.internetcomputer.org/wiki/Current_limitations_of_the_Internet_Computer), section "Calling potentially malicious or buggy canisters can prevent canisters from upgrading"

### Make sure there are no loops in call graphs

#### Security Concern

Loops in the call graph (e.g. canister A calling B, B calling C, C calling A) may lead to canister deadlocks.

#### Recommendation

-   Avoid such loops!

-   For more information, see [Current limitations of the Internet Computer](https://wiki.internetcomputer.org/wiki/Current_limitations_of_the_Internet_Computer), section "Loops in call graphs"

## Canister Upgrades

### Be careful with panics during upgrades

#### Security Concern

If a canister traps or panics in `pre_upgrade`, this can lead to permanently blocking the canister, resulting in a situation where upgrades fail or are no longer possible at all.

#### Recommendation

-   Avoid panics / traps in `pre_upgrade` hooks, unless it is truly unrecoverable, so that any invalid state can fixed by upgrading. Panics in the pre-upgrade hook prevent upgrade, and since the pre-upgrade hook is controlled by the old code, it can permanently block upgrading.

-   Panic in the `post_upgrade` hook if state is invalid, so that one can retry the upgrade and try to fix the invalid state. Panics in the the post-upgrade hook abort the upgrade, but one can retry with new code.

-   [Test the upgrade hooks.](https://mmapped.blog/posts/01-effective-rust-canisters.html#test-upgrades) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html))

-   See also the section on upgrades in [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister) (though focused on Motoko)

-   See [Current limitations of the Internet Computer](https://wiki.internetcomputer.org/wiki/Current_limitations_of_the_Internet_Computer), section "Bugs in `pre_upgrade` hooks"

## Miscellaneous

### Test your canister code even in presence of System API calls

#### Security Concern

Since canisters interact with the system API, it is harder to test the code because unit tests cannot call the system API. This may lead to lack of unit tests.

#### Recommendation

-   Create loosely coupled modules that do not depend on the system API and unit test those. See this [recommendation](https://mmapped.blog/posts/01-effective-rust-canisters.html#target-independent) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html)).

-   For the parts that still interact with the system API: create a thin abstraction of the System API that is faked in unit tests. See the [recommendation](https://mmapped.blog/posts/01-effective-rust-canisters.html#target-independent) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html)). For example, one can implement a ‘Runtime’ as follows and then use the ‘MockRuntime’ in tests (code by Dimitris Sarlis):

<!-- -->

        use ic_cdk::api::{
            call::call, caller, data_certificate, id, print, time, trap,
        };

        #[async_trait]
        pub trait Runtime {
            fn caller(&self) -> Result<Principal, String>;
            fn id(&self) -> Principal;
            fn time(&self) -> u64;
            fn trap(&self, message: &str) -> !;
            fn print(&self, message: &str);
            fn data_certificate(&self) -> Option<Vec<u8>>;
            (...)
        }

        #[async_trait]
        impl Runtime for RuntimeImpl {
            fn caller(&self) -> Result<Principal, String> {
                let caller = caller();
                // The anonymous principal is not allowed to interact with the canister.
                if caller == Principal::anonymous() {
                    Err(String::from(
                        "Anonymous principal not allowed to make calls.",
                    ))
                } else {
                    Ok(caller)
                }
            }

            fn id(&self) -> Principal {
                id()
            }

            fn time(&self) -> u64 {
                time()
            }

            (...)

        }

        pub struct MockRuntime {
            pub caller: Principal,
            pub canister_id: Principal,
            pub time: u64,
            (...)
        }

        #[async_trait]
        impl Runtime for MockRuntime {
            fn caller(&self) -> Result<Principal, String> {
                Ok(self.caller)
            }

            fn id(&self) -> Principal {
                self.canister_id
            }

            fn time(&self) -> u64 {
                self.time
            }

            (...)

        }

### Make canister builds reproducible

#### Security Concern

It should be possible to verify that a canister does what it claims to do. the IC provides a SHA256 hash of the deployed WASM module. In order for this to be useful, the canister build has to be reproducible.

#### Recommendation

Make canister builds reproducible. See this [recommendation](https://mmapped.blog/posts/01-effective-rust-canisters.html#reproducible-builds) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html)). See also [Developer docs on this](../../developer-docs/build/backend/reproducible-builds).

### Expose metrics from your canister

#### Security Concern

In case of attacks, it is great to be able to obtain relevant metrics from canisters, such as number of accounts, size of internal data structures, stable memory, etc.

#### Recommendation

[Expose metrics from your canister.](https://mmapped.blog/posts/01-effective-rust-canisters.html#expose-metrics) (from [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html))

### Don’t rely on time being strictly monotonic

#### Security Concern

The time read from the System API is monotonic, but not strictly monotonic. Thus, two subsequent calls can return the same time, which could lead to security bugs when the time API is used.

#### Recommendation

See the "Time is not strictly monotonic" section in [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister)

### Protect against draining the cycles balance

#### Security Concern

Canisters pay for their cycles which makes them inherently vulnerable to attacks that consume all their cycles.

#### Recommendation

Consider monitoring, early authentication, rate limiting on canister level to mitigate this. Also, be aware that an attacker will aim for the call consuming most cycles. See the "Cycle balance drain attacks section" in [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister) .

## Nonspecific to the Internet Computer

The best practices in this section are very general and not specific to the Internet Computer. This list is by no means complete and only lists a few very specific concerns that have led to issues in the past.

### Validate inputs

#### Security Concern

The data sent in [query and update calls](../ic-interface-spec#http-interface) is generally untrusted. The message size limit is a few MB. This can e.g. lead the following issues:

-   If unvalidated data is rendered in web UIs or displayed in other systems, this can lead to injection attacks (e.g. XSS).

-   Messages of big size could be sent and potentially stored in the canister, consuming an excessive amount of storage.

-   Big inputs (e.g. big lists or strings) could trigger an excessive amount of computation, resulting in DoS and consuming many cycles. See also [Protect against draining the cycles balance](#protect-against-draining-the-cycles-balance)

#### Recommendation

-   Perform input validation, see e.g. the [OWASP cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html).

-   "Large data attacks" section in [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister) (be aware of Candid space bombs)

-   [ASVS](https://owasp.org/www-project-application-security-verification-standard/) 5.1.4: Verify that structured data is strongly typed and validated against a defined schema including allowed characters, length and pattern (e.g. credit card numbers or telephone, or validating that two related fields are reasonable, such as checking that suburb and zip/postcode match).

### Rust: Don’t use unsafe Rust code

#### Security Concern

Unsafe Rust code is risky because it may introduce memory corruption issues.

#### Recommendation

-   Avoid unsafe code whenever possible.

-   See the [Rust security guidelines](https://anssi-fr.github.io/rust/04_language.html#unsafe-code)

-   Consider the [Dfinity Rust Guidelines](https://docs.dfinity.systems/dfinity/spec/meta/rust.html#_avoid_unsafe_code).

### Rust: Avoid integer overflows

#### Security Concern

Integers in Rust may overflow. While such overflows lead to panics in the debug configuration, the values are just wrapped around silently in release compilation. This can cause major security issues e.g. when the integers are used as indices, unique IDs, or if cycles or ICP amounts are computed.

#### Recommendation

-   Review your code carefully for any integer operations that may wrap around.

-   Use the `saturated` or `checked` variants of these operations, such as `saturated_add`, `saturated_sub`, `checked_add` , `checked_sub`, etc. See e.g. the [Rust docs](https://doc.rust-lang.org/std/primitive.u32.html#method.saturating_add) for `u32`.

-   See also the [Rust security guidelines on integer overflows](https://anssi-fr.github.io/rust/04_language.html#integer-overflows).

### For expensive calls, consider using captchas or proof of work

#### Security Concern

If an update or query call is expensive e.g. in terms of memory used or cycles consumed, this may make it easy for bots to render the canister unusable (e.g. by filling up it’s storage).

#### Recommendation

If the dApp offers such operations, consider bot prevention techniques such as adding Captchas or proof of work. There is e.g. a captcha implementation in [internet identity](https://github.com/dfinity/internet-identity).
