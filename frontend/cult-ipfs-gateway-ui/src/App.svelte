<script>
  import ValueCreatorForm from "./components/ValueCreatorForm.svelte";
  import Asset from "./components/Asset.svelte";
  import Seo from "./Seo.svelte";
  import { onMount } from "svelte";
  import { backendBaseURL } from "./stores";
  import Metamask from "./components/Metamask.svelte";
  import Invite from "./components/invitations/Invite.svelte";
  // import Invite from "./components/invitations/Invite.svelte";
  // import Web3 from "web3";

  let assets = [];
  let filteredassets = [];
  let showInviteForm = false;
  let searchTerm = "";
  let typingActive = false;
  let showValueCreatorForm = false;
  let showQualifyForBeingInvitedInstructions = false;
  let showPhilosophy = false;
  let publicWalletAddress = "";
  let web3;
  let host;

  onMount(async () => {
    getAssetsFromServer();
    setTimeout(() => {
      // optimizing speed for first meaningful content display ... https://pagespeed.web.dev/
      getInvites();
    }, 1000 * 1);
  });

  const getAssetsFromServer = async () => {
    const urlToGetassets = `${backendBaseURL}/api/v1/getAssets`;
    console.log(`fetching assets from ${urlToGetassets}`);
    const response = await fetch(urlToGetassets);

    assets = await response.json();
    filteredassets = [...assets];
  };

  const getInvites = async () => {
    const urlToGetInvitesFormatted = `${backendBaseURL}/api/v1/getInvites`;
    console.log(`fetching invites from ${urlToGetInvitesFormatted}`);
    const response = await fetch(urlToGetInvitesFormatted);

    host = await response.json();
  };

  const handleNewAsset = () => {
    getAssetsFromServer();
    alert("Asset added successfully. Thank you for supporting the CULT.");
    showValueCreatorForm = false;
  };

  const handleWalletConnected = async (event) => {
    publicWalletAddress = event.detail.publicWalletAddress;
    web3 = event.detail.web3;
  };

  const changeShowQualifyForBeingInvitedInstructions = () => {
    showQualifyForBeingInvitedInstructions =
      !showQualifyForBeingInvitedInstructions;
    if (showQualifyForBeingInvitedInstructions) {
      showInviteForm = false;
      showValueCreatorForm = false;
      showPhilosophy = false;
    }
  };
  const changeShowValueCreatorForm = () => {
    showValueCreatorForm = !showValueCreatorForm;
    if (showValueCreatorForm) {
      showInviteForm = false;
      showQualifyForBeingInvitedInstructions = false;
      showPhilosophy = false;
    }
  };

  const changeShowInviteForm = () => {
    showInviteForm = !showInviteForm;
    if (showInviteForm) {
      showValueCreatorForm = false;
      showQualifyForBeingInvitedInstructions = false;
      showPhilosophy = false;
    }
  };

  const changeShowPhilosophy = () => {
    showPhilosophy = !showPhilosophy;
    if (showPhilosophy) {
      showValueCreatorForm = false;
      showQualifyForBeingInvitedInstructions = false;
      showInviteForm = false;
    }
  };


  const onKeyDown = () => {
    filteredassets = [...assets];
    if (typingActive === false) {
      typingActive = true;

      setTimeout(() => {
        const currentFilterResult = [];
        for (const asset of filteredassets) {
          const stringifiedasset = JSON.stringify(asset);
          if (stringifiedasset.indexOf(searchTerm) !== -1) {
            currentFilterResult.push(asset);
          }
        }

        filteredassets = [...currentFilterResult];
        typingActive = false;
      }, 1000 * 1);
    }
  };

  const isWalletInvited = (walletAddress) => {
    const stringifiedInvitationTree = JSON.stringify(host);
    if (
      stringifiedInvitationTree.indexOf(walletAddress) === -1 &&
      host.length !== undefined
    ) {
      return false;
    }
    return true;
  };
</script>

<Seo
  title="CULT IPFS Gateway"
  description="We are a network of cultdao.io fans promoting freedom, fairness, education and love."
/>

<main class="container">
  <div class="text-center">
    <!-- <h1>CULT Assets Explorer</h1> -->
    <h1>CULT IPFS Gateway</h1>

    <p><br /><br /></p>

    Here you can 
    <a
      href="#addEducationAsset"
      class="whiteLink"
      on:click={changeShowValueCreatorForm}
    >
      upload
    </a>
    CULT images & diagrams to the 
    <a href="https://ipfs.tech" target="_blank" class="champagne">IPFS</a>. <br />

    Each upload gets a link which you can use on 
    <a href="https://cultdonations.org" target="_blank" class="champagne">cultdonations.org</a>.
    <p><br /><br /></p>

    <div class="input-group">
      <!-- svelte-ignore a11y-autofocus -->
      <input
        type="searchTerm"
        bind:value={searchTerm}
        placeholder="... start typing to become a cult explorer"
        on:keydown={onKeyDown}
        autofocus
      />
    </div>

    <p><br /></p>
    Number of Results: {filteredassets.length}
    <p><br /></p>

    <div class="assetsArea">
      {#each filteredassets as asset}
        <Asset {asset} />
      {/each}
    </div>

    <p><br /></p>

    <!-- <section id="addEducationAsset">
      <button on:click={() => changeShowValueCreatorForm()}> Add Link </button>
      {#if showValueCreatorForm}
        <Metamask
          on:walletConnected={handleWalletConnected}
          showConnectedWallet={false}
        />

        {#if isWalletInvited(publicWalletAddress)}
          <ValueCreatorForm
            on:newAsset={handleNewAsset}
            {web3}
            {publicWalletAddress}
          />
        {:else}
          <Invite
            showExplanation={true}
            showInvitationsTree={true}
            connectedWallet={publicWalletAddress}
            {host}
          />
        {/if}
      {/if}
    </section>

    <p><br /></p>

    <section id="invitations">
      <button on:click={() => changeShowInviteForm()}> Invite Friends </button>
      {#if showInviteForm}
        <Metamask
          on:walletConnected={handleWalletConnected}
          showConnectedWallet={false}
        />

        {#if isWalletInvited(publicWalletAddress)}
          <Invite
            showInviteForm={true}
            showInvitationsTree={true}
            connectedWallet={publicWalletAddress}
            {host}
          />
        {:else}
          <Invite
            showExplanation={true}
            showInvitationsTree={true}
            connectedWallet={publicWalletAddress}
            {host}
          />
        {/if}
      {/if}
    </section>

    <p><br /></p>

    <section id="qualifyForBeingInvited">
      <button on:click={() => changeShowQualifyForBeingInvitedInstructions()}>
        Qualify For Being Invited
      </button>
      {#if showQualifyForBeingInvitedInstructions}
      <p><br /></p>
        Every wallet can invite maximum 5 further wallets. <br />
        If you are not invited yet, you might consider quote tweeting:
        <a
          href="https://twitter.com/Peer2peerE/status/1582845604329582593?s=20&t=uwn0dTKHXYbjVx2ZxzR5Jg"
          target="_blank"
          class="champagne"
        >
          this tweet</a
        >
        and adding a public wallet address which you would like to use to receive
        donations -
        <a
          href="https://twitter.com/Peer2peerE/status/1582863692403593216?s=20&t=uwn0dTKHXYbjVx2ZxzR5Jg"
          target="_blank"
          class="champagne">example</a
        >.
        <p><br></p>
      {/if}
    </section>
    <p><br></p> -->
    <!-- <section id="philosophy">
      <button on:click={() => changeShowPhilosophy()}> Philosophy </button>
      {#if showPhilosophy}
      <p><br></p>
      Nobody can prevent us humans from giving something to someone for free. <br
      />
      Let's create a system where people donate valuable things to each other.
      <br />
      There will be a mechanism to reward those who donate - leveraging
      quadratic donations (inspired by quadratic funding) and or recursive donations along the invitation tree-branch.
      <p><br></p>
      {/if}
    </section> -->
    <p><br></p>
    <a href="https://github.com/michael-spengler/cult-ipfs-gateway-microservice/issues/new" target="_blank">
      <button> Give Feedback </button>
    </a>
  </div>
</main>

<style>
  .champagne {
    color: #efdcb3;
  }

  .assetsArea {
    max-height: 140vh;
    overflow-y: scroll;
  }

  ::-webkit-scrollbar {
    display: none;
  }
  .whiteLink {
    color: #efdcb3;
  }
  h1 {
    color: #d7c69d;
  }
</style>
