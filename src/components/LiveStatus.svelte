<script>
  import Loading from "../components/Loading.svelte";
  import { onMount } from "svelte";
  import config from "../data/config.json";
  import { createOctokit, handleError } from "../utils/createOctokit";

  let loading = true;
  const octokit = createOctokit();
  const owner = config.owner;
  const repo = config.repo;
  let { apiBaseUrl } = config["status-website"] || {};
  let sites = [];
  if (!apiBaseUrl) apiBaseUrl = "https://api.github.com";
  const userContentBaseUrl = apiBaseUrl.includes("api.github.com")
    ? `https://raw.githubusercontent.com`
    : apiBaseUrl;
  const graphsBaseUrl = `${userContentBaseUrl}/${owner}/${repo}/master/graphs`;

  onMount(async () => {
    try {
      sites = JSON.parse(
        atob(
          (
            await octokit.repos.getContent({
              owner,
              repo,
              path: "history/summary.json",
            })
          ).data.content.replace(/\n/g, "")
        )
      );
    } catch (error) {
      handleError(error);
    }
    loading = false;
  });
</script>

<style>
  article {
    background-size: contain;
    background-position: top right;
    background-repeat: no-repeat;
  }
</style>

<h2>{config.i18n.liveStatus}</h2>
<section>
  {#if loading}
    <Loading />
  {:else if sites.length}
    {#each sites as site}
      <article
        class={`${site.status} link`}
        style={`background-image: url("${graphsBaseUrl}/${site.slug}.png`}>
        <h4><a href={`${config.path}/history/${site.slug}`}>{site.name}</a></h4>
        <div>
          {@html config.i18n.overallUptime.replace(/\$UPTIME/g, site.uptime)}
        </div>
        <div>
          {@html config.i18n.averageResponseTime.replace(/\$TIME/g, site.time)}
        </div>
      </article>
    {/each}
  {/if}
</section>
