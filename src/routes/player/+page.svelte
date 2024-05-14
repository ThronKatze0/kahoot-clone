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
  import { getAuth, signInAnonymously } from "firebase/auth";
  import {
    DocumentReference,
    Timestamp,
    doc,
    getDoc,
    getFirestore,
    setDoc,
    updateDoc,
  } from "firebase/firestore";
  import { Doc, SignedIn, SignedOut, docStore, userStore } from "sveltefire";
  // import { clearInterval } from "timers";

  initializeStores();
  const toastStore = getToastStore();
  const modalStore = getModalStore();

  const app = initializeApp(/* your firebase config */);
  const auth = getAuth(app);
  const user = userStore(auth);
  const firestore = getFirestore(app);

  $: userDocReference =
    $user != null ? doc(firestore, "/users/" + $user!.uid) : null;

  $: userDoc =
    $user != null ? docStore(firestore, "/users/" + $user!.uid) : null;

  $: gameFile =
    $user != null && $userDoc != null
      ? doc(firestore, "/games/" + $userDoc.gameCode + "/" + $user!.uid)
      : null;

  $: gameDoc =
    $userDoc != null
      ? docStore(firestore, "/games/" + $userDoc!.gameCode)
      : null;

  console.log($user);
  console.log($userDoc);

  let gameCodeInput: string = "000000";

  let selectedOption: boolean = false;
  let option: number = 0;

  function triggerToast(message: string, style: string) {
    let styleString: string;
    switch (style) {
      case "success":
        styleString = "variant-filled-success";
        break;
      case "warning":
        styleString = "variant-filled-warning";
        break;
      default:
        styleString = "variant-filled-error";
        break;
    }
    toastStore.trigger({
      message: message,
      background: styleString,
    });
  }

  async function joinGame() {
    let game = doc(firestore, "/games/" + gameCodeInput);
    getDoc(game)
      .then((gameDoc) => {
        if (gameDoc.exists()) {
          modalStore.trigger({
            type: "prompt",
            title: "Choose your username",
            value: "",
            valueAttr: {
              type: "text",
              minlength: 3,
              maxlength: 18,
              required: true,
            },
            response: (r: string) => {
              signInAnonymously(auth)
                .then(() => {
                  setDoc(doc(firestore, "/users/" + $user?.uid), {
                    username: r,
                    score: 0,
                    selectedOption: 0,
                    gameCode: gameCodeInput,
                  });
                  setDoc(
                    doc(
                      firestore,
                      "/games/" + gameCodeInput + "/players/" + $user?.uid,
                    ),
                    {
                      username: r,
                      score: 0,
                      selectedOption: 0,
                    },
                  );
                })
                .catch((error) => {
                  console.log(error);
                });
            },
          });
        } else {
          triggerToast("Game doesn't exist", "error");
          gameCodeInput = "";
        }
      })
      .catch((error) => {
        triggerToast(error, "error");
      });
  }

  function selectOption(
    paramOption: number,
    rightOption: number,
    questionStartTimestamp: Timestamp,
  ) {
    // TODO: shouldn't be hardcoded

    // let plusScore: number = 1000 - () * 50;
    let plusScore = 1000;
    console.log(paramOption, rightOption);
    if (paramOption == rightOption) {
      updateDoc(gameFile!, {
        selectedOption: paramOption,
        score: $userDoc?.score + plusScore,
      });
    } else {
      updateDoc(gameFile!, {
        selectedOption: paramOption,
      });
    }
    selectedOption = true;
    option = paramOption;
  }
</script>

<Toast />
<Modal />

{#if $user == null}
  <div class="flex flex-col justify-center items-center min-h-screen w-full">
    <h1 class="h1 text-center mb-5">Enter Game Code</h1>
    <div class="flex flex-row justify-center items-center w-full">
      <input
        type="text"
        class="input w-1/3 mr-4"
        placeholder="Game Code"
        bind:value={gameCodeInput}
      />
      <button
        class="btn variant-filled"
        on:click={() => {
          joinGame();
        }}>Join</button
      >
    </div>
  </div>
{:else}
  <Doc ref={$gameDoc} let:data>
    {#if data.state == "notStarted"}
      <div
        class="flex flex-col justify-center items-center min-h-screen w-full"
      >
        <h1 class="h1 text-center">You're in! Wait until the game starts...</h1>
      </div>
    {:else if data.state == "questionRead"}
      <div
        class="flex flex-col justify-center items-center min-h-screen w-full"
      >
        <h1 class="h1 text-center mb-5">
          Read the question on the big screen...
        </h1>
        <ProgressRadial width="w-1/4" />
      </div>
    {:else if data.state == "optionSelect"}
      {#if selectedOption}
        <div
          class="flex flex-col justify-center items-center min-h-screen w-full"
        >
          <h1 class="h1 text-center mb-5">
            Option submitted! Let's wait for the others to choose...
          </h1>
          <ProgressRadial width="w-1/4" />
        </div>
      {:else}
        <div
          class="grid grid-cols-2 grid-rows-2 gap-2 w-full min-w-full min-h-screen p-2"
        >
          <button
            on:click={() => {
              selectOption(0, data.currentAnswer, data.lastQuestionStart);
            }}
            class="bg-red-700 hover:bg-red-800 transition-all duration-200 rounded flex justify-center items-center"
          >
            <Icon icon="uim:square-shape" width="50" height="50" />
          </button>
          <button
            on:click={() => {
              selectOption(1, data.currentAnswer, data.lastQuestionStart);
            }}
            class="bg-blue-700 hover:bg-blue-800 transition-all duration-200 rounded flex justify-center items-center"
          >
            <Icon
              icon="fluent-mdl2:circle-shape-solid"
              width="50"
              height="50"
            />
          </button>
          <button
            on:click={() => {
              selectOption(2, data.currentAnswer, data.lastQuestionStart);
            }}
            class="bg-yellow-700 hover:bg-yellow-800 transition-all duration-200 rounded flex justify-center items-center"
          >
            <Icon
              icon="fluent-mdl2:triangle-shape-solid"
              width="50"
              height="50"
            />
          </button>
          <button
            on:click={() => {
              selectOption(3, data.currentAnswer, data.lastQuestionStart);
            }}
            class="bg-green-700 hover:bg-green-800 transition-all duration-200 rounded flex justify-center items-center"
          >
            <Icon icon="gis:regular-shape" width="60" height="60" />
          </button>
        </div>
      {/if}
    {:else if data.state == "selectFinished"}
      {#if option == data.currentAnswer}
        <div
          class="flex flex-col justify-center items-center min-h-screen w-full"
        >
          <Icon
            icon="healthicons:yes-outline"
            color="#2ddf16"
            width="300"
            height="300"
          />
          <h1 class="h1 text-center mb-5">
            Congratulations! You picked the right option!
          </h1>
        </div>
      {:else}
        <div
          class="flex flex-col justify-center items-center min-h-screen w-full"
        >
          <Icon
            icon="healthicons:no-outline"
            color="#dc1818"
            width="300"
            height="300"
          />
          <h1 class="h1 text-center mb-5">You picked the wrong option</h1>
        </div>
      {/if}
    {/if}
  </Doc>
{/if}
