<script>
    import Auth from "../components/Auth.svelte";

    export let qs;

    qs.action = qs.action || "register"
    if (!["login", "register"].includes(qs.action)) qs.action = "register"
</script>

<h1 class="w3-center w3-jumbo">Authenticate to F80 DATA</h1>
<Auth authMode={qs.action} on:auth={() => window.location.href=qs.next || "/"} />