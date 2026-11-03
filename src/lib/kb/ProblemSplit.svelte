<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem found in a merged ticket, each linked to its own issue.
  let { ticket, issues }: { ticket: number; issues: Record<string, string> } = $props();

  // Counted the desk's way: by tenant and channel (causes.ts).
  const problemOf = (r: Report) => causeOf(r, tenantOf(r)) ?? 'unplaced';

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = $derived(
    Object.entries(
      merged.reduce<Record<string, number>>((n, r) => ({ ...n, [problemOf(r)]: (n[problemOf(r)] ?? 0) + 1 }), {}),
    ).sort((a, b) => b[1] - a[1]),
  );
</script>

<p><strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.length}</strong> problems</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>ticket</th></tr></thead>
  <tbody>
    {#each rows as [problem, count] (problem)}
      <tr>
        <td><code>{problem}</code></td>
        <td>{count}</td>
        <td>{#if issues[problem]}<a href={issues[problem]}>{issues[problem].split('/').slice(-3).join(' ')}</a>{:else}none yet{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each rows as [problem] (problem)}
  {@const moved = merged.filter((r) => problemOf(r) === problem && notes.oneDayMove.includes(r.id)).length}
  {#if moved}
    <p class="note">{moved} of the <code>{problem}</code> reports read as a one-day move (desk notes); counted under <code>{problem}</code> by tenant.</p>
  {/if}
{/each}

<style>
  .note { color: var(--muted); font-size: 0.85em; }
</style>
