<script>
  import Loading from "../components/Loading.svelte";
  import { onMount } from "svelte";
  import config from "../data/config.json";
  import { cachedResponse, createOctokit, handleError } from "../utils/createOctokit";

  let loading = true;
  const octokit = createOctokit();
  const owner = config.owner;
  const repo = config.repo;
  let incidents = [];

  onMount(async () => {
    try {
      incidents = (
        await cachedResponse(`issues-${owner}-${repo}`, () =>
          octokit.issues.listForRepo({
            owner,
            repo,
            state: "open",
            filter: "all",
            sort: "created",
            direction: "desc",
            labels: "status",
          })
        )
      ).data;
      incidents = incidents.map((incident, index) => {
        incident.showHeading =
          index === 0 ||
          new Date(incidents[index - 1].created_at).toLocaleDateString() !==
            new Date(incident.created_at).toLocaleDateString();
        return incident;
      });
    } catch (error) {
      handleError(error);
    }
    loading = false;
  });
</script>

{#if !incidents.length && !loading}
  <article class="up"><svg xmlns="http://www.w3.org/2000/svg" width="1.6em" height="1.6em" viewBox="0 0 24 24" fill="none">
<path fill-rule="evenodd" clip-rule="evenodd" d="M12 22C17.5228 22 22 17.5228 22 12C22 6.47715 17.5228 2 12 2C6.47715 2 2 6.47715 2 12C2 17.5228 6.47715 22 12 22ZM10.2426 14.4142L17.3137 7.34315L18.7279 8.75736L10.2426 17.2426L6 13L7.41421 11.5858L10.2426 14.4142Z" fill="#B0DCA2FF"/>
</svg>&nbsp; {config.i18n.allSystemsOperational}</article>
{/if}

<section>
  {#if loading}
    <Loading />
  {:else if incidents.length}
    <h2>{config.i18n.activeIncidents}</h2>
    {#each incidents as incident}
      <article
        class="down down-active link {incident.title.includes('degraded') ? 'degraded' : ''}"
      >
        <div class="f">
          <div>
            <h4>{incident.title.replace("🛑", "").replace("⚠️", "").trim()}</h4>
            <div>
              {config.i18n.activeIncidentSummary
                .replace(/\$DATE/g, new Date(incident.created_at).toLocaleString(config.i18n.locale))
                .replace(/\$POSTS/g, incident.comments)}
            </div>
          </div>
          <div class="f r">
            <a href={`${config.path}/incident/${incident.number}`}>
              {config.i18n.incidentReport.replace(/\$NUMBER/g, incident.number)}
            </a>
          </div>
        </div>
      </article>
    {/each}
  {/if}
</section>

<style>
   section {
     margin-bottom: 2rem;
   }
   article.up {
     display: flex;
     align-items: center;
     gap: 0.5rem;
   }
 </style>
