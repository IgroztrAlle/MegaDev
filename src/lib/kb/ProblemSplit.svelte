<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  // The desk-notes reports describe a one-day move, not a month/day swap: a third problem.
  const shifted = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)).length);
  const rows = $derived([
    ...groups.map((g) => ({ problem: g.cause, reports: g.reports.length })),
    { problem: 'one-day shift', reports: shifted },
  ].sort((a, b) => b.reports - a.reports));
</script>

<table>
  <thead><tr><th>problem</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr><td><code>{row.problem}</code></td><td>{row.reports}</td></tr>
    {/each}
  </tbody>
  <tfoot><tr><td>total merged into #{ticket}</td><td>{merged.length}</td></tr></tfoot>
</table>
<p class="note">Counted as the desk files them: the {shifted} one-day-shift reports are also inside the parser count, so the rows add up to more than the total.</p>

<style>
  .note { color: var(--muted); font-size: 0.85em; }
</style>
