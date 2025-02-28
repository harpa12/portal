# Default Cycles Wallet

As discussed in [Tokens and cycles](../../../concepts/tokens-cycles), ICP tokens can be converted into **cycles** to power canister operations. Cycles reflect the operational cost of communication, computation, and storage that dapps consume.

Unlike ICP tokens, cycles are only associated with canisters and not user or developer principals. Because only canisters require and consume cycles—to perform operations and to pay for the resources they use—users and developers manage the distribution and ownership of cycles through a special type of canister called a **cycles wallet**. Because the cycles wallet holds the cycles required to perform operations such as creating new canisters, these operations are executed using the canister principal for the cycles wallet instead of your user principal.

For the purposes of using the local canister execution environment, the SDK automatically creates a default cycles wallet for you in every project and most of the operations performed using the cycles wallet happen behind the scenes. For example, the cycles wallet acts on your behalf to register canister principals and deploy canisters in the local canister execution environment.

In a production environment, however, you need to explicitly register and transfer cycles to new canisters, specify the principals that can act as custodians, and manage the principals with ownership rights. You can perform some of these tasks using the default cycles wallet dapp running in a web browser. Depending on the specific action you want to take, you can also perform these cycle and canister management tasks by running `dfx wallet` commands in a terminal or by calling methods in the default cycles wallet canister directly.

You should keep in mind, however, that calls to the cycles wallet canister are executed using the cycles wallet principal associated with the currently-selected user identity. Depending on your currently-selected identity and whether the principal associated with that identity has been added as a controller or a custodian for a wallet, you might see different results or be denied access to a specific method.

To check the identity you are currently using, run the following command:

    dfx identity whoami

## Controller and Custodian Roles

A user principal or canister principal can be assigned to a **controller** or **custodian** role.

A **controller** is the most privileged role and a principal assigned to the controller role can perform privileged tasks including the following:

-   Add and remove other principals as controllers.

-   Authorize and de-authorize other principals as custodians.

-   Add entries to the cycles wallet address book.

-   Access the cycles wallet balance and all other wallet-related information.

-   Send cycles to other canisters.

-   Accept receipt of cycles from other canisters.

-   Rename the cycles wallet.

-   Create canisters and additional cycles wallets.

A principal assigned to the **custodian** role can only perform a subset of cycles wallet management tasks, including the following:

-   Access the cycles wallet balance and all other wallet-related information.

-   Send cycles to other canisters.

-   Accept receipt of cycles from other canisters.

-   Create canisters.

Authorizing a principal as a custodian does not automatically grant the principal access to a cycles wallet. The identity assigned to the custodian role must also be assigned a cycles wallet principal. For example, if you authorize the identity `alice_custodian` as a custodian of a cycles wallet (`rwlgt-iiaaa-aaaaa-aaaaa-cai`) in a local project, that user would also need to be assigned to use that wallet with the `dfx identity set-wallet rwlgt-iiaaa-aaaaa-aaaaa-cai` command.

## Create a Cycles Wallet

If you are doing local development, your cycles wallet is created when you register a new canister principal using `dfx canister create` or when you register, build, and deploy a canister with `dfx deploy`.

If you are deploying on the Internet Computer, you typically create your cycles wallet by converting ICP tokens to cycles, transferring the cycles to a new canister principal, and updating the canister with the default cycles wallet WebAssembly module (WASM).

There are dapps that can help you convert ICP to cycles and create a new cycles wallet, e.g., [NNS dapp](../../../tokenomics/token-holders/nns-app-quickstart#_deploy_a_canister_with_cycles).

## Check the Cycle Balance

In the local canister execution environment or with a cycles wallet on the Internet Computer, you can use the `dfx wallet balance` command or the `wallet_balance` method to check the current cycle balance.

### Check your cycles balance when developing locally

If you are doing local development, you can use the `dfx wallet balance` command to check the current cycles balance on a project-by-project basis.

To check the cycles balance in a local project:

1.  Open a terminal and navigate to the root directory of the project.

2.  Start the local canister execution environment by running the following command:

        dfx start --background

3.  Display the cycles balance from the cycles wallet associated with the currently-selected identity by running the following command:

        dfx wallet balance

    The command displays output similar to the following:

        78000000000000 cycles.

### Check the cycles balance on the Internet Computer

If you have deployed a cycles wallet on the Internet Computer, you can use the `dfx wallet balance` command to check the current cycles balance on the network.

To check the cycles balance on the Internet Computer:

1.  Open a terminal and navigate to a directory that contains a `dfx.json` configuration file.

2.  Check your connection to the Internet Computer by running the following command:

        dfx ping ic

3.  Display the cycle balance from the cycles wallet associated with the currently-selected identity by running the following command:

        dfx wallet --network ic balance

    The command displays output similar to the following:

        67991783875995 cycles.

### Call the cycles wallet\_balance method

You can also check the cycles balance by calling the `wallet_balance` method in the cycles wallet canister directly. For example, if your principal is a controller for the `h5aet-waaaa-aaaab-qaamq-cai` cycles wallet, you can check the current cycle balance by running the following command:

    dfx canister --networkiccall h5aet-waaaa-aaaab-qaamq-cai wallet_balance

The command returns the balance using Candid format as a record with an amount field (represented by the hash 3\_573\_748\_184) and a balance of 6,895,656,625,450 cycles like this:

    (record { 3_573_748_184 = 6_895_656_625_450 })

## Add a Controller

If you are the controller of a cycles wallet, you can add other user principals or canister principals to the controller role. Adding a principal to the controller role also automatically adds the principal to the custodian role.

To add a controller to a cycles wallet in a the local project:

1.  Open a terminal and navigate to the root directory of the project.

2.  Start the local canister execution environment by running the following command:

        dfx start --background

3.  Display the cycles balance from the cycles wallet associated with the currently-selected identity by running a command similar to the following:

        dfx wallet add-controller <controller-principal>

    For example, you would run the following command to add the user represented by the principal b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe as a controller of the local cycles wallet:

        dfx wallet add-controller b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe

    The command displays output similar to the following:

        Added b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe as a controller.

## List the Current Controllers

You can use the `dfx wallet controllers` command or the `get_controllers` method to list the principals that have full control over a specified cycles wallet canister.

To list the controllers for a cycles wallet in a local project:

1.  Open a terminal and navigate to the root directory of the project.

2.  Start the local canister execution environment by running the following command:

        dfx start --background

3.  List the principals that have full control over the cycles wallet in the current project by running the following command:

        dfx wallet controllers

    The command displays the textual representation of the principals that have control over the cycles wallet with output similar to the following:

        tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe
        b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe

## Remove a Controller

You can use the `dfx wallet remove-controller` command or the `remove_controller` method to remove a principal as a controller.

To remove a controller for a cycles wallet in a local project:

1.  Open a terminal and navigate to the root directory of the project.

2.  Start the local canister execution environment by running the following command:

        dfx start --background

3.  Specify the principal to remove from the controller role in the current project by running a command similar to the following:

        dfx wallet remove-controller b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe

    The command output similar to the following:

        Removed b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe as a controller.

## Authorize a Custodian

You can use the `dfx wallet authorize` command or the `authorize` method to authorize a principal as a custodian of a cycles wallet.

To authorize a principal as a custodian for the cycles wallet in a local project:

1.  Open a terminal and navigate to the root directory of the project.

2.  Start the local canister execution environment by running the following command:

        dfx start --background

3.  Specify the principal to authorize as a custodian in the current project and for the current identity by running a command similar to the following:

        dfx wallet authorize b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe

    The command output similar to the following:

        Authorized b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe as a custodian.

## List Current Custodians

You can use the `dfx wallet custodians` command or the `get_custodians` method to return the list of principals that are currently defined as custodians for the cycles wallet.

To list the custodians for a cycles wallet in a local project:

1.  Open a terminal and navigate to the root directory of the project.

2.  Start the local canister execution environment by running the following command:

        dfx start --background

3.  List the principals that have the custodian role for the cycles wallet in the current project by running the following command:

        dfx wallet custodians

    The command displays output similar to the following:

        tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe
        b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe

## Remove Authorization for a Custodian

You can use the `dfx wallet deauthorize` command or the `deauthorize` method to remove a principal as a custodian for a cycles wallet. De-authorizing a principal that was previously added as a controller also automatically removes the principal from the controller role.

To remove a custodian for a cycles wallet in a local project:

1.  Open a terminal and navigate to the root directory of the project.

2.  Start the local canister execution environment by running the following command:

        dfx start --background

3.  Specify the principal to remove from the custodian role in the current project by running a command similar to the following:

        dfx wallet deauthorize b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe

    The command output similar to the following:

        Deauthorized b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe as a custodian.

## Send Cycles to a Canister

You can use `dfx wallet send` command of the `wallet_send` method to send a specific number of cycles to a specific canister. Keep in mind that the canister you specify must be a cycles wallet or have a `wallet_receive` method to accept the cycles.

If you have deployed a cycles wallet on the Internet Computer, you can use the `dfx wallet send` command to send cycles between canisters.

To send cycles to another canister running on the Internet Computer:

1.  Open a terminal and navigate to a directory that contains a `dfx.json` configuration file.

2.  Check your connection to the Internet Computer by running the following command:

        dfx ping ic

3.  Get the principal for the canister that you want to receive the cycles.

    For example, run the following command to display the cycles wallet principal associated with the current user identity on the Internet Computer:

        dfx identity --networkicget-wallet

    The command displays the cycles wallet principal with output similar to the following:

        gastn-uqaaa-aaaae-aaafq-cai

4.  Send cycles to the canister by running a command similar to the following:

        dfx wallet --networkicsend <destination> <amount>

    For example:

        dfx wallet --networkicsend gastn-uqaaa-aaaae-aaafq-cai 10000000000

    If the transfer is successful, the command does not displays any output.

    The maximum number of cycles that can be stored in a cycles wallet is 2<sup>128</sup>.

5.  Check the cycles wallet balance to see the updated number of cycles available by running the following command:

        dfx wallet --network ic balance

    For example:

        67991699387090 cycles.

## List Address Book Entries

You can use the `dfx wallet addresses` command or the `list_addresses` method to list the principals and roles that have been configured for the cycles wallet.

To view address book entries for a cycles wallet running on the Internet Computer:

1.  Open a terminal and navigate to a directory that contains a `dfx.json` configuration file.

2.  Check your connection to the Internet Computer by running the following command:

        dfx ping ic

3.  Get the address book entries for the cycles wallet by running the following command :

        dfx wallet --networkicaddresses

    The command displays the controllers and custodians for the cycles wallet with output similar to the following:

        Id: tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe, Kind: Unknown, Role: Controller, Name: No name set.
        Id: ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe, Kind: Unknown, Role: Custodian, Name: No name set.
        Id: b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe, Kind: Unknown, Role: Controller, Name: No name set.

## Additional Methods in the Default Cycles Wallet

The default cycles wallet canister includes additional methods that are not exposed as `dfx wallet` commands. The additional methods support more advanced cycles management tasks such as creating new canisters and managing events.

### Create a new cycles wallet

Use the `wallet_create_wallet` method to create a new cycles wallet canister with an initial cycle balance and, optionally, with a specific principal as its controller. If you don’t specify a controlling principal, the cycles wallet you use to create the new wallet will be the new wallet’s controller.

For example, you can run a command similar to the following to create a new wallet and assign a principal as a controller:

    dfx canister --network  call f3yw6-7qaaa-aaaab-qaabq-cai wallet_create_wallet '(record { cycles = 5000000000000 : nat64; controller = principal "vpqee-nujda-46rtu-4noo7-qnxmb-zqs7g-5gvqf-4gy7t-vuprx-u2urx-gqe"})'

The command returns the principal for the new wallet:

    (record { 1_313_628_723 = principal "dcxxq-jqaaa-aaaab-qaavq-cai" })

### Register a new canister principal

Use the `wallet_create_canister` method to register a new canister principal on the Internet Computer. This method creates a new "empty" canister placeholder with an initial cycle balance and, optionally, with a specific principal as its controller. After you have registered the canister principal, you can install code for your canister as a separate step.

For example, you can run a command similar to the following to create a new wallet and assign a principal as a controller:

    dfx canister --network  call f3yw6-7qaaa-aaaab-qaabq-cai wallet_create_canister '(record { cycles = 5000000000000 : nat64; controller = principal "vpqee-nujda-46rtu-4noo7-qnxmb-zqs7g-5gvqf-4gy7t-vuprx-u2urx-gqe"})'

The command returns the principal for the new canister you created:

    (record { 1_313_628_723 = principal "dxqg5-iyaaa-aaaab-qaawa-cai" })

### Receive cycles from a canister

Use the `wallet_receive` method as an endpoint to receive cycles.

### Forward calls from a wallet

Use the `wallet_call` method to forward calls using the cycles wallet principal as caller.

### Manage addresses

Use the following methods to manage address book entries:

-   `add_address`: (address: AddressEntry) → ();

-   `remove_address`: (address: principal) → ();

### Manage events

Use the following methods to retrieve event and chart information.

-   `get_events`: (opt record { from: opt nat32; to: opt nat32; }) → (vec Event) query;

-   `get_chart`: (opt record { count: opt nat32; precision: opt nat64; } ) → (vec record { nat64; nat64; }) query;

For example, you can use the `get_events` method to return `canister_create` and other events by running a command similar to the following:

    dfx canister call <cycles-wallet-principal> get_events '(record {from = null; to = null})'

If the cycles wallet (`gastn-uqaaa-aaaae-aaafq-cai`) is deployed on the Internet Computer main network, you could run a command that looks like this to return events:

    dfx canister --networkiccall gastn-uqaaa-aaaae-aaafq-cai get_events '(record {from = null; to = null})'

The output from the command is in Candid format similar to the following:

    (
      vec { record { 23_515 = 0; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe"; 1_224_700_491 = null; 1_269_754_742 = variant { 4_218_395_836 };} }; 2_781_795_542 = 1_621_456_688_636_513_683;}; record { 23_515 = 1; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe"; 1_224_700_491 = null; 1_269_754_742 = variant { 2_494_206_670 };} }; 2_781_795_542 = 1_621_461_468_638_569_551;}; record { 23_515 = 2; 1_191_829_844 = variant { 1_205_528_161 = record { 2_190_693_645 = 11_000_000_000_000; 2_631_180_839 = principal "gvvca-vyaaa-aaaae-aaaga-cai";} }; 2_781_795_542 = 1_621_462_573_993_647_258;}; record { 23_515 = 3; 1_191_829_844 = variant { 1_205_528_161 = record { 2_190_693_645 = 11_000_000_000_000; 2_631_180_839 = principal "gsueu-yaaaa-aaaae-aaagq-cai";} }; 2_781_795_542 = 1_621_462_579_193_578_440;}; record { 23_515 = 4; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "install_code"; 2_631_180_839 = principal "aaaaa-aa";} }; 2_781_795_542 = 1_621_462_593_047_590_026;}; record { 23_515 = 5; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "install_code"; 2_631_180_839 = principal "aaaaa-aa";} }; 2_781_795_542 = 1_621_462_605_779_157_885;}; record { 23_515 = 6; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "authorize"; 2_631_180_839 = principal "gsueu-yaaaa-aaaae-aaagq-cai";} }; 2_781_795_542 = 1_621_462_609_036_146_536;}; record { 23_515 = 7; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "greet"; 2_631_180_839 = principal "gvvca-vyaaa-aaaae-aaaga-cai";} }; 2_781_795_542 = 1_621_463_144_066_333_270;}; record { 23_515 = 8; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe"; 1_224_700_491 = null; 1_269_754_742 = variant { 2_494_206_670 };} }; 2_781_795_542 = 1_621_463_212_828_477_570;}; record { 23_515 = 9; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "wallet_balance"; 2_631_180_839 = principal "gastn-uqaaa-aaaae-aaafq-cai";} }; 2_781_795_542 = 1_621_878_637_071_884_946;}; record { 23_515 = 10; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe"; 1_224_700_491 = null; 1_269_754_742 = variant { 4_218_395_836 };} }; 2_781_795_542 = 1_621_879_473_916_547_313;}; record { 23_515 = 11; 1_191_829_844 = variant { 313_999_214 = record { 1_136_829_802 = principal "gastn-uqaaa-aaaae-aaafq-cai"; 3_573_748_184 = 10_000_000_000;} }; 2_781_795_542 = 1_621_977_470_023_492_664;}; record { 23_515 = 12; 1_191_829_844 = variant { 2_171_739_429 = record { 25_979 = principal "gastn-uqaaa-aaaae-aaafq-cai"; 3_573_748_184 = 10_000_000_000; 4_293_698_680 = 0;} }; 2_781_795_542 = 1_621_977_470_858_839_320;};},
    )

In this example, there are twelve event records. The Role field (represented by the hash `1_269_754_742`) specifies whether a principal is a controller (represented by the hash `4_218_395_836`) or a custodian (represented by the hash `2_494_206_670`). The events in this example also illustrate an amount field (represented by the hash `3_573_748_184`) with a transfer of 10,000,000,000 cycles.
