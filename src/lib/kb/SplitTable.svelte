<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem in a merged ticket. The cause rules in causes.ts place every report by
  // tenant; reports the desk read as a one-day move (desk-notes.json) are not a month flip, so they
  // get a row of their own instead of a footnote.
  let { ticket }: { ticket: number } = $props();

  type Row = { problem: string; what: string; reports: Report[] };

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = $derived.by((): Row[] => {
    const shifted = merged.filter((r) => notes.oneDayMove.includes(r.id));
    const flipped = merged.filter((r) => !notes.oneDayMove.includes(r.id));
    const what: Record<string, string> = {
      parser: 'shared parser reads day-first dates month first',
      hint: 'booking-v3 form hint says MM/DD/YYYY, the form reads the tenant’s order',
      unplaced: 'no tenant on file, or the tenant writes month first',
    };
    return [
      ...groupByCause(flipped, tenantOf).map((g) => ({ problem: g.cause, what: what[g.cause], reports: g.reports })),
      ...(shifted.length ? [{ problem: 'one-day', what: 'date moves by one day, not a flip', reports: shifted }] : []),
    ];
  });
  const tenants = (rs: Report[]) => new Set(rs.map((r) => r.tenant)).size;
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>what breaks</th><th>reports</th><th>tenants</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr>
        <td><code>{row.problem}</code></td>
        <td>{row.what}</td>
        <td>{row.reports.length}</td>
        <td>{tenants(row.reports)}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
