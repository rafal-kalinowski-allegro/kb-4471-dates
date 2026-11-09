<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const inGroup = (cause: string) => groups.find((g) => g.cause === cause)?.reports ?? [];
  // One-day moves are a third problem: the desk notes single them out, but cause-by-tenant files them under parser.
  const isMove = (r: Report) => notes.oneDayMove.includes(r.id);
  const rows = $derived([
    { problem: '1 · shared parser reads dates month first (includes the 5 of problem 3, as the desk files them)', n: inGroup('parser').length },
    { problem: '2 · booking-v3 form hints MM/DD/YYYY to day-first tenants', n: inGroup('hint').length },
    { problem: '3 · bookings land one day off (time zone or clock, not a month flip; also counted in 1)', n: merged.filter(isMove).length },
  ]);
</script>

<p><strong>{merged.length}</strong> unique reports in #{ticket} · <strong>{rows.length}</strong> problems</p>
<table>
  <thead><tr><th>problem</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as r (r.problem)}<tr><td>{r.problem}</td><td>{r.n}</td></tr>{/each}
  </tbody>
</table>
