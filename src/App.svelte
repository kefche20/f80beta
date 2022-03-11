<script>
    import router from "page";
    import Index from "./routes/index.svelte";
    import Auth from "./routes/auth.svelte";
	import Chat from "./routes/chat.svelte";
    import NotFound from "./routes/notfound.svelte";
    import { queryString } from "./services/util";

    let page;
    let params = {};
	let qs = {};

    router("/", () => page = Index)
	router("/404/:msg", function (ctx, next) {
    params = ctx.params;
    next();
}, function () { return page = NotFound; });

    router("/auth", (ctx, next) => {
        qs = queryString(ctx.querystring)
        next();
    }, () => page = Auth)
    router("/login", () => router.redirect("/auth"))
    router("/register", () => router.redirect("/auth"))
	router("/chat", () => page = Chat)
    router("/*", () => page = NotFound)
    router.start();
</script>

<svelte:component this={page} {params} {qs} />