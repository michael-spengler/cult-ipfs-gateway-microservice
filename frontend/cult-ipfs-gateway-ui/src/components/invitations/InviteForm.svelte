<script>
    import Metamask from "../Metamask.svelte";
    import { backendBaseURL } from "../../stores";
    import { isEthereumWalletAddress } from "../../helpers";

    // import { createEventDispatcher } from "svelte";
    // const dispatch = createEventDispatcher();
    let walletToBeInvited = "";
    let signature = "";
    let publicWalletAddress = "";
    let web3;
    let infoMessageToBeSigned =
        "This signature is used to validate that you are the owner of this wallet.";

    let message = "";
    const invite = async () => {
        if (isEthereumWalletAddress(walletToBeInvited.toLowerCase())) {
            try {
                signature = await web3.eth.sign(
                    web3.utils.toHex(infoMessageToBeSigned),
                    publicWalletAddress
                );
            } catch (error) {
                console.log(error);
            }

            if (signature !== "") {
                try {
                    const inviteWalletURL = `${backendBaseURL}/api/v1/inviteWallet`;
                    console.log(`register invite via ${inviteWalletURL}`);
                    const inviteInfo = {
                        host: publicWalletAddress,
                        signature,
                        invitees: [{ host: walletToBeInvited, invitees: [] }],
                    };
                    const response = await fetch(inviteWalletURL, {
                        method: "post",
                        headers: {
                            Accept: "application/json",
                            "Content-Type": "application/json",
                        },
                        body: JSON.stringify(inviteInfo),
                    });

                    
                    message = (await response.json()).message;
                    walletToBeInvited = "";
                } catch (error) {
                    alert(`an error occurred: ${error.message}`);
                }
            } else {
                alert("Without signature you cannot invite a wallet. This is to ensure there is no ground for bad actors in our system.")
            }
        } else {
            alert(
                `${walletToBeInvited} seems not to be a valid Ethereum Address`
            );
        }
    };

    const handleWalletConnected = async (event) => {
        publicWalletAddress = event.detail.publicWalletAddress;
        web3 = event.detail.web3;
    };

    const clickRefresh = () => {
        window.location.reload();
    };
</script>

<h2>Invite Your Friends</h2>
<br />
{#if message !== ""}
    <div class="message">
        {message}
        <p><br /></p>
        <button class="button-colors-on-Card" on:click={clickRefresh}>
            Refresh Page
        </button>
        <p />
    </div>
{/if}
{#if message === ""}
    <div class="input-group">
        <input
            type="text"
            bind:value={walletToBeInvited}
            placeholder="... paste invitee wallet ..."
        />
    </div>
    <p />
{/if}
{#if walletToBeInvited !== ""}
    <p><br /></p>
    <div class="color-of-body">
        <button class="button-colors-on-Card" on:click={() => invite()}
            >Send</button
        >
        <p><br /></p>
        <Metamask
            on:walletConnected={handleWalletConnected}
            showConnectedWallet={true}
        />
    </div>
{/if}
