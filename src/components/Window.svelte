<script>
    import {invoke} from "@tauri-apps/api";
    import PokemonScreen from "./main/PokemonScreen.svelte";
    import StartUpScreen from "./updater/StartUpScreen.svelte";
    import UpdateManager from "./updater/UpdateManager.svelte";
    import {checkUpdate} from "@tauri-apps/api/updater";
    import {onMount} from "svelte";

    // Load options from file
    let options;
    let showUpdateScreen = null;

    onMount(async () => {
        const reload = async (afterReload) => {
            await invoke("get_options").then(async (result) => {
                options = result;

                // Debug options - might be interesting to see what's in there
                console.debug("read options", options);

                // Easy way to store options
                options.store = async function () {
                    console.debug("storing options", options);
                    await invoke("store_options", {options}).catch(e => console.error(e));
                };

                options.reload = reload;

                options.toggleTheme = function () {
                    if (options.theme === "LIGHT") {
                        options.theme = "DARK";
                        window.document.body.classList.add("dark-mode");
                    } else {
                        options.theme = "LIGHT";
                        window.document.body.classList.remove("dark-mode");
                    }
                    invoke("store_options", {options}).catch(e => console.error(e));
                };

                if (options.theme === "DARK") {
                    window.document.body.classList.add("dark-mode");
                } else {
                    window.document.body.classList.remove("dark-mode");
                }

                try {
                    const {shouldUpdate} = await checkUpdate();
                    showUpdateScreen = shouldUpdate;
                    invoke("console_log_info", {message: `Checking for Updates... ${shouldUpdate}`}).catch(e => console.error(e));
                } catch (error) {
                    showUpdateScreen = false;
                    invoke("console_log_error", {message: `error`}).catch(e => console.error(e));
                }

            });
        }
        reload()
    });


    invoke("check_online_status").then((result) => {
        console.debug("online status", result);
    }).catch(e => {
        alert("You are offline! Please connect to the internet and restart the app.\n If this problem persists, please contact the developer.\n\n (Error: " + e + ")");
        console.error(e);
    });
</script>

<div class="window">
    {#if showUpdateScreen !== null && options !== null}
        {#if showUpdateScreen}
            <UpdateManager/>
        {:else}
            <PokemonScreen bind:options={options}></PokemonScreen>
        {/if}
    {:else}
        <StartUpScreen/>
    {/if}
</div>
