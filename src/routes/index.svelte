
<script>
	import { onMount } from 'svelte';

  import {flip} from 'svelte/animate';
  import {dndzone} from 'svelte-dnd-action';

  import ogSquareImageSrc from '$lib/assets/home/home-open-graph-square.jpg';
  import ogImageSrc from '$lib/assets/home/home-open-graph.jpg';
  import twitterImageSrc from '$lib/assets/home/home-twitter.jpg';
  import featuredImageSrc from '$lib/assets/home/home.jpg';
  //import BlogRoll from '$lib/components/BlogRoll.svelte';
  //import Card from '$lib/components/Card.svelte';
  import SEO from '$lib/components/SEO/index.svelte';
  import website from '$lib/config/website';

  const { author, siteUrl } = website;

  let title = 'Home';
  const breadcrumbs = [
    {
      name: 'Home',
      slug: '',
    },
  ];
  let metadescription = '';
  const featuredImageAlt = '';
  const featuredImage = {
    url: featuredImageSrc,
    alt: featuredImageAlt,
    width: 672,
    height: 448,
    caption: 'Home page',
  };
  const ogImage = {
    url: ogImageSrc,
    alt: featuredImageAlt,
  };
  const ogSquareImage = {
    url: ogSquareImageSrc,
    alt: featuredImageAlt,
  };

  const twitterImage = {
    url: twitterImageSrc,
    alt: featuredImageAlt,
  };
  const entityMeta = {
    url: `${siteUrl}/`,
    faviconWidth: 512,
    faviconHeight: 512,
    caption: author,
  };
  const seoProps = {
    title,
    slug: '',
    entityMeta,
    datePublished: '2021-07-07T14:19:33.000+0100',
    lastUpdated: '2021-07-07T14:19:33.000+0100',
    breadcrumbs,
    metadescription,
    featuredImage,
    ogImage,
    ogSquareImage,
    twitterImage,
  };

  
  //let Moralis;
  /* Moralis init code */
  const serverUrl = 'https://up4dw9h6ggww.usemoralis.com:2053/server';
  const appId = 'vq18qD9kJiueOdtWq6qzQXPwTCuRpHRCjxYD0sRh';

  $:userNFTs = [];

  onMount(async() => {
    //Moralis = await import('moralis/dist/moralis.min.js');
    
    await Moralis.start({ serverUrl, appId });
    //autologin
    login();

  });

  /**
   * login
   */
  async function login() {
    console.log('[login]')
    let user = Moralis.User.current();
    console.log('[login]',user)
    if (!user) {
      user = await Moralis.authenticate({ signingMessage: "Log in to The IMR Bridge via the Moralis SDK" })
        .then(function (user) {
          console.log("logged in user:", user);
          console.log(user.get("ethAddress"));
        })
        .catch(function (error) {
          console.log(error);
        });
    }
  }
  
  /**
   * logOut
   */
  async function logOut() {
    await Moralis.User.logOut();
    console.log("logged out");
  }

  /**
   * getNFTs
   */
  async function getNFTs() {
    const userEthNFTs = await Moralis.Web3API.account.getNFTs({ chain: 'rinkeby' });
    console.log('userEthNFTs',userEthNFTs.result);
    const hasNFTs = userEthNFTs.result.filter((nft) => {
      if (nft.metadata !== null) {
        nft.metadata = JSON.parse(nft.metadata);
      }
      return (nft.metadata !== null);
    });
    console.log('hasNFTs',hasNFTs)
    userNFTs = hasNFTs;
  }


  /**
   * getImg
   */
  function getImg(path) {
    if (path.startsWith('ipfs')) {
        let splitPath = path.replace(/ipfs:\/\//g,'').split('/');
        //console.log(splitPath);
        path = `https://${splitPath[0]}.ipfs.infura-ipfs.io/${splitPath[1]}`
    }

    return path;
  }
</script>

<style>
  .interlinkedItems {
    margin:0px;
    padding:4px 8px;
    border:solid 1px #ccc;
    margin-bottom:20px;
    border-radius: 8px;
  }
  .interlinkedItems li {
    display:flex;
    list-style: none;
    margin:0px;
  }
  .interlinkedItems li > div {
    flex:1;
    border:solid 1px #ccc;
    border-radius: 8px;
    margin:4px 8px;
    overflow: hidden;
    padding:10px;
  }
  .interactionOptions {
    display: flex;
  }
  .interactionOptions select {
    flex:1;
  }
  .header {
    text-align:center;
    font-weight:bold;
  }
</style>


<SEO {...seoProps} />


<button on:click="{login}">login</button>
<button on:click="{logOut}">logout</button>
<button on:click="{getNFTs}">getNFTs</button>


<header>
  <h1>Metaverse Interlinked Wallet</h1>
</header>

<ul>
  {#each userNFTs as nft}
    <li>
      <img src="{getImg(nft.metadata.image)}" alt=""/>
      {nft.metadata.name}
    </li>
  {/each}
</ul>