<script>
    import App from "./fb";
    import { onMount } from "svelte";
    import { getAuth, onAuthStateChanged } from "firebase/auth";
    import { goto } from "$app/navigation";
    import { page } from "$app/stores";

    onMount(() => {
        const auth = getAuth();
        onAuthStateChanged(auth, (user) => {
            let path = $page.url.pathname;
            if (!user) {
                if (path === "/home") {
                    goto("/");
                }
            } else {
                if (path === "/login" || path === "/signup") {
                    goto("/home");
                }
            }
        });
    });
</script>

<slot />