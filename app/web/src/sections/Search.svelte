<script>
  import { TOKEN } from "../api.js";

  let mode = $state("jobs"); // jobs | people
  let keywords = $state("");
  let location = $state("");
  let company = $state("");
  let loading = $state(false);
  let error = $state(null);
  let raw = $state(null);

  async function run() {
    loading = true; error = null; raw = null;
    const path = mode === "jobs" ? "jobs" : "people";
    const body =
      mode === "jobs"
        ? { keywords, location }
        : { keywords, company, location };
    try {
      const res = await fetch(`/api/search/${path}?token=${encodeURIComponent(TOKEN)}`, {
        method: "POST",
        headers: { "content-type": "application/json" },
        body: JSON.stringify(body),
      });
      const json = await res.json();
      if (!res.ok) throw new Error(json.error ?? `HTTP ${res.status}`);
      raw = json;
    } catch (e) {
      error = e.message;
    } finally {
      loading = false;
    }
  }

  // The MCP returns a sections object with a free-text results blob; surface it.
  let resultText = $derived(
    raw?.sections?.search_results ?? raw?.raw ?? (raw ? JSON.stringify(raw, null, 2) : null)
  );
  let resultRefs = $derived(raw?.references?.search_results ?? []);
</script>

<h1 class="page-title">Search</h1>
<p class="page-sub">Instant LinkedIn search · "deep scout" hand-off to the agent comes in Phase 3</p>

<div class="panel">
  <div style="display:flex;gap:8px;margin-bottom:14px">
    <button class="tab {mode === 'jobs' ? 'on' : ''}" onclick={() => (mode = 'jobs')}>Jobs</button>
    <button class="tab {mode === 'people' ? 'on' : ''}" onclick={() => (mode = 'people')}>Recruiters / People</button>
  </div>

  <div style="display:grid;grid-template-columns:1fr 1fr 1fr auto;gap:10px;align-items:end">
    <label>Keywords<input bind:value={keywords} placeholder={mode === 'jobs' ? 'software engineer' : 'recruiter engineer Brazil'} /></label>
    {#if mode === 'people'}
      <label>Company<input bind:value={company} placeholder="Airbnb" /></label>
    {/if}
    <label>Location<input bind:value={location} placeholder="São Paulo" /></label>
    <button class="go" onclick={run} disabled={loading || !keywords}>
      {loading ? "Searching…" : "Search"}
    </button>
  </div>
</div>

{#if error}
  <div class="panel" style="border-color:var(--red)">
    <strong style="color:var(--red)">Search failed.</strong>
    <p class="muted" style="font-size:.85rem;margin-top:6px">{error}</p>
    <p class="dim" style="font-size:.8rem;margin-top:6px">
      Make sure the LinkedIn MCP is reachable and you're logged in to LinkedIn in your browser session.
    </p>
  </div>
{:else if raw}
  <div class="panel">
    <h3>Results</h3>
    {#if resultRefs.length}
      <div style="margin-bottom:12px;display:flex;flex-wrap:wrap;gap:8px">
        {#each resultRefs.filter((r) => r.kind === "person" || r.kind === "job") as ref}
          <a class="ref" href={`https://www.linkedin.com${ref.url}`} target="_blank">{ref.text ?? ref.url}</a>
        {/each}
      </div>
    {/if}
    <pre class="results">{resultText}</pre>
  </div>
{/if}

<style>
  label { display:flex; flex-direction:column; gap:6px; font-size:.78rem; color:var(--dim); }
  input {
    background:var(--panel-2); border:1px solid var(--border-2); border-radius:8px;
    padding:9px 11px; color:var(--text); font-size:.9rem;
  }
  input:focus { outline:none; border-color:var(--blue); }
  .tab {
    background:var(--panel-2); border:1px solid var(--border-2); color:var(--muted);
    padding:7px 14px; border-radius:8px; cursor:pointer; font-size:.85rem;
  }
  .tab.on { background:var(--blue); color:white; border-color:var(--blue); }
  .go {
    background:linear-gradient(135deg,var(--blue),var(--cyan)); color:white; border:none;
    padding:9px 18px; border-radius:8px; cursor:pointer; font-weight:600; font-size:.9rem;
  }
  .go:disabled { opacity:.5; cursor:not-allowed; }
  .ref { background:var(--panel-2); border:1px solid var(--border-2); border-radius:7px; padding:4px 10px; font-size:.8rem; }
  .results {
    background:var(--panel-2); border:1px solid var(--border); border-radius:8px;
    padding:14px; font-size:.82rem; line-height:1.5; color:var(--muted);
    white-space:pre-wrap; max-height:520px; overflow:auto; font-family:ui-monospace,monospace;
  }
</style>
