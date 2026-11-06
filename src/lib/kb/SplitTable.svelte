<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const count = (cause: string) => groups.find((g) => g.cause === cause)?.reports.length ?? 0;
  // One-day moves stay counted under `parser`, where the desk files them; footnoted below.
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rows = $derived([
    { problem: 'parser reads dates month first', n: count('parser') },
    { problem: 'booking-v3 form hints MM/DD/YYYY to day-first tenants', n: count('hint') },
  ]);
</script>

<p><strong>{merged.length}</strong> unique reports in #{ticket} · <strong>{rows.length}</strong> problems</p>
<table>
  <thead><tr><th>problem</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as r (r.problem)}<tr><td>{r.problem}</td><td>{r.n}</td></tr>{/each}
  </tbody>
</table>
<p class="note">{oneDay.length} of the parser reports read as a one-day move, not a month flip; counted under parser, as the desk files them.</p>
