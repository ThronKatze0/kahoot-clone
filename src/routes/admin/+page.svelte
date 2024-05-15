<script lang="ts">
  import Icon from "@iconify/svelte";
  import {
    Modal,
    ProgressRadial,
    Toast,
    getModalStore,
    getToastStore,
    initializeStores,
  } from "@skeletonlabs/skeleton";
  import { initializeApp } from "firebase/app";
  import {
    getAuth,
    onAuthStateChanged,
    signInAnonymously,
  } from "firebase/auth";
  import {
    DocumentReference,
    Timestamp,
    doc,
    getDoc,
    getFirestore,
    setDoc,
    updateDoc,
  } from "firebase/firestore";
  import {
    Collection,
    Doc,
    SignedIn,
    SignedOut,
    docStore,
    userStore,
  } from "sveltefire";

  initializeStores();
  const toastStore = getToastStore();
  const modalStore = getModalStore();

  const app = initializeApp(/* your firebase config */);
  const auth = getAuth(app);
  const user = userStore(auth);
  const firestore = getFirestore(app);

  let gameCode: string = "000000";
  $: gameRef = "/games/" + gameCode;
</script>

<Doc ref={gameRef} let:data>
  {#if data.state == "notStarted"}
    <h1 class="h1 mt-10 text-center">Industrielle Revolution</h1>
    <h2 class="h2 mt-20 text-center">{gameCode}</h2>
    <Collection ref={gameRef + "/players"} let:data let:count>
      <div class="grid grid-rows-8 gap-4">
        {#each data as player}
          <div class="flex justify-between items-center">
            <div class="flex items-center">
              <Icon icon="mdi:account"></Icon>
              <p class="ml-2">{player.name}</p>
            </div>
          </div>
        {/each}
      </div></Collection
    >
  {:else}
    <!-- else content here -->
  {/if}
</Doc>
