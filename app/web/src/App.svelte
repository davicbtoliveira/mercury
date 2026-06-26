<script>
  import { api, onConnection } from "./api.js";
  import { resource } from "./lib/resource.svelte.js";
  import { useLiveTable } from "./lib/live.svelte.js";
  import Overview from "./sections/Overview.svelte";
  import Recruiters from "./sections/Recruiters.svelte";
  import Jobs from "./sections/Jobs.svelte";
  import Search from "./sections/Search.svelte";
  import Launch from "./sections/Launch.svelte";
  import Profile from "./sections/Profile.svelte";
  import Applications from "./sections/Applications.svelte";
  import Interviews from "./sections/Interviews.svelte";
  import Activity from "./sections/Activity.svelte";
  import {
    LayoutDashboard, UserRound, Search as SearchIcon, Rocket, Users,
    Briefcase, FileText, CalendarClock, Activity as ActivityIcon,
  } from "@lucide/svelte";

  let active = $state("overview");
  let connected = $state(false);

  // Overview drives the sidebar badges + the Overview page. Refresh it when any
  // table that feeds its counts changes.
  const overview = resource(() => api("overview"), null);
  useLiveTable(
    ["recruiters", "jobs", "interviews", "applications", "profile_metrics"],
    overview.reload
  );

  $effect(() => onConnection((c) => (connected = c)));

  const nav = [
    { id: "overview", label: "Overview", icon: LayoutDashboard },
    { id: "profile", label: "Profile", icon: UserRound },
    { id: "search", label: "Search", icon: SearchIcon },
    { id: "launch", label: "Launch", icon: Rocket },
    { id: "recruiters", label: "Recruiters", icon: Users },
    { id: "jobs", label: "Jobs", icon: Briefcase },
    { id: "applications", label: "Applications", icon: FileText },
    { id: "interviews", label: "Interviews", icon: CalendarClock },
    { id: "activity", label: "Activity", icon: ActivityIcon },
  ];

  let ov = $derived(overview.status === "ready" ? overview.data : null);
</script>

<div class="grid grid-cols-[220px_1fr] min-h-screen">
  <aside class="relative bg-panel border-r border-border px-3.5 py-[22px] sticky top-0 h-screen">
    <div class="grad text-2xl font-extrabold mb-7 pl-2">Mercury</div>
    {#each nav as item}
      {@const Icon = item.icon}
      <button
        class="w-full flex items-center gap-2.5 px-3 py-2.5 rounded-[9px] text-[0.92rem] font-medium border
               {active === item.id
                 ? 'bg-panel-2 text-text border-border-2'
                 : 'text-muted border-transparent hover:bg-panel-2 hover:text-text'}"
        onclick={() => (active = item.id)}
      >
        <Icon size={17} strokeWidth={2} />
        <span>{item.label}</span>
        {#if item.id === "recruiters" && ov}<span class="ml-auto text-[0.72rem] text-dim">{ov.recruiters}</span>{/if}
        {#if item.id === "interviews" && ov}<span class="ml-auto text-[0.72rem] text-dim">{ov.interviews}</span>{/if}
        {#if item.id === "jobs" && ov}<span class="ml-auto text-[0.72rem] text-dim">{ov.jobs}</span>{/if}
      </button>
    {/each}
    <div class="absolute bottom-[18px] left-3.5 text-[0.74rem] text-dim flex items-center gap-1.5">
      <span class="w-2 h-2 rounded-full inline-block"
        style:background={connected ? "var(--color-green)" : "var(--color-dim)"}
        style:box-shadow={connected ? "0 0 8px var(--color-green)" : "none"}></span>
      {connected ? "live" : "offline"}
    </div>
  </aside>

  <main class="px-9 pt-[30px] pb-20 max-w-[1180px]">
    {#if active === "overview"}<Overview {overview} onnav={(id) => (active = id)} />
    {:else if active === "profile"}<Profile />
    {:else if active === "search"}<Search />
    {:else if active === "launch"}<Launch />
    {:else if active === "recruiters"}<Recruiters />
    {:else if active === "jobs"}<Jobs />
    {:else if active === "applications"}<Applications />
    {:else if active === "interviews"}<Interviews />
    {:else if active === "activity"}<Activity />
    {/if}
  </main>
</div>
