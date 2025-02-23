<script lang="ts">
  import { sum } from "lodash";
  import { DIRECTIONS } from "src/constants";
  import type { Directions, HierData } from "src/interfaces";
  import type BreadcrumbsPlugin from "src/main";
  import {
    closeImpliedLinks,
    copy,
    debug,
    hierToStr,
    mergeGs,
  } from "src/sharedFunctions";

  export let plugin: BreadcrumbsPlugin;

  const { settings } = plugin;
  const { userHierarchies } = settings;
  const separator = settings.trailSeperator;

  const hierGs = plugin.currGraphs;

  function fillInInfo(
    dir: Directions,
    gType: string,
    hierData: HierData,
    nodesToo = true
  ) {
    const g = hierData[dir][gType].graph;

    if (nodesToo) {
      const nodes = hierData[dir][gType].graph.nodes();
      hierData[dir][gType].nodes = nodes;
      hierData[dir][gType].nodesStr = nodes.join("\n");
    }

    const edges = hierData[dir][gType].graph.edges();
    hierData[dir][gType].edges = edges;
    let edgeStrArr = edges.map(
      (e) =>
        `${nodesToo ? e.v : e.w} ${
          nodesToo || dir !== "same" ? separator : "⟷"
        } ${nodesToo ? e.w : e.v}`
    );
    hierData[dir][gType].edgesStr = edgeStrArr.join("\n");
  }

  const data = hierGs.hierGs.map((hier) => {
    const hierData: HierData = {
      up: { Merged: {}, Closed: {}, Implied: {} },
      same: { Merged: {}, Closed: {}, Implied: {} },
      down: { Merged: {}, Closed: {}, Implied: {} },
    };
    DIRECTIONS.forEach((dir) => {
      // Merged Graphs
      /// Smoosh all fieldGs from one dir into a merged graph for that direction as a whole
      const mergedInDir = mergeGs(...Object.values(hier[dir]));
      hierData[dir].Merged.graph = mergedInDir;
      fillInInfo(dir, "Merged", hierData);

      // Closed graphs
      if (dir !== "same") {
        hierData[dir].Closed.graph = closeImpliedLinks(
          mergedInDir,
          mergeGs(...Object.values(hier[dir === "up" ? "down" : "up"]))
        );
      } else {
        hierData[dir].Closed.graph = closeImpliedLinks(
          mergedInDir,
          mergedInDir
        );
      }
      fillInInfo(dir, "Closed", hierData);

      if (dir !== "same") {
        hierData[dir].Implied.graph = mergeGs(
          ...Object.values(hier[dir === "up" ? "down" : "up"])
        );
      } else {
        hierData[dir].Implied.graph = closeImpliedLinks(
          mergedInDir,
          mergedInDir
        );
      }
      fillInInfo(dir, "Implied", hierData, false);
    });

    return hierData;
  });

  debug(settings, { data });

  let hierStrs: string[] = userHierarchies.map(hierToStr);

  //   const [pIEdgesStr, sIEdgesStr, cIEdgesStr] = [
  //     complement(
  //       pAEdges.map((e) => `${e.v} ${separator} ${e.w}`),
  //       pREdges.map((e) => `${e.v} ${separator} ${e.w}`)
  //     ).join("\n"),
  //     complement(
  //       sAEdges.map((e) => `${e.v} ${separator} ${e.w}`),
  //       sREdges.map((e) => `${e.v} ${separator} ${e.w}`)
  //     ).join("\n"),
  //     complement(
  //       cAEdges.map((e) => `${e.v} ${separator} ${e.w}`),
  //       cREdges.map((e) => `${e.v} ${separator} ${e.w}`)
  //     ).join("\n"),
  //   ];
</script>

<table>
  <thead>
    <tr>
      <th scope="col">Hierarchy</th>
      <th scope="col" colspan="5">Count</th>
    </tr>
  </thead>

  <tr>
    <td />
    <td>Measure</td>
    <td>↑</td>
    <td>→</td>
    <td>↓</td>
    <td>Total</td>
  </tr>

  {#each userHierarchies as hier, i}
    <tr>
      <td rowspan="3">
        {hierStrs[i]}
      </td>
      <td>Nodes</td>
      {#each ["up", "same", "down"] as dir}
        <td
          aria-label={data[i][dir].Merged.nodesStr}
          on:click={async () => await copy(data[i][dir].Merged.nodesStr)}
        >
          {data[i][dir].Merged.nodes.length}
        </td>
      {/each}
      <td
        aria-label={[
          data[i].up.Merged.nodesStr,
          data[i].same.Merged.nodesStr,
          data[i].down.Merged.nodesStr,
        ].join("\n")}
        on:click={async () =>
          await copy(
            [
              data[i].up.Merged.nodesStr,
              data[i].same.Merged.nodesStr,
              data[i].down.Merged.nodesStr,
            ].join("\n")
          )}
      >
        {[
          ...data[i].up.Merged.nodes,
          ...data[i].same.Merged.nodes,
          ...data[i].down.Merged.nodes,
        ].length}
      </td>
    </tr>
    <tr>
      <td>Real Edges</td>
      {#each ["up", "same", "down"] as dir}
        <td
          aria-label={data[i][dir].Merged.edgesStr}
          on:click={async () => await copy(data[i][dir].Merged.edgesStr)}
        >
          {data[i][dir].Merged.edges.length}
        </td>
      {/each}
      <td
        aria-label={[
          data[i].up.Merged.edgesStr,
          data[i].same.Merged.edgesStr,
          data[i].down.Merged.edgesStr,
        ].join("\n")}
        on:click={async () =>
          await copy(
            [
              data[i].up.Merged.edgesStr,
              data[i].same.Merged.edgesStr,
              data[i].down.Merged.edgesStr,
            ].join("\n")
          )}
      >
        {[
          ...data[i].up.Merged.edges,
          ...data[i].same.Merged.edges,
          ...data[i].down.Merged.edges,
        ].length}
      </td>
    </tr>
    <tr>
      <td>Implied Edges</td>
      {#each ["up", "same", "down"] as dir}
        <td
          aria-label={data[i][dir].Implied.edgesStr}
          on:click={async () => await copy(data[i][dir].Implied.edgesStr)}
        >
          {data[i][dir].Implied.edges.length}
        </td>
      {/each}
      <td
        aria-label={[
          data[i].up.Implied.edgesStr,
          data[i].same.Implied.edgesStr,
          data[i].down.Implied.edgesStr,
        ].join("\n")}
        on:click={async () =>
          await copy(
            [
              data[i].up.Implied.edgesStr,
              data[i].same.Implied.edgesStr,
              data[i].down.Implied.edgesStr,
            ].join("\n")
          )}
      >
        {[
          ...data[i].up.Implied.edges,
          ...data[i].same.Implied.edges,
          ...data[i].down.Implied.edges,
        ].length}
      </td>
    </tr>
  {/each}
  <tr>
    <td rowspan="3"> Totals </td>
    <td>Nodes</td>
    {#each ["up", "same", "down"] as dir}
      <td
        aria-label={data.map((datum) => datum[dir].Merged.nodesStr).join("\n")}
        on:click={async () =>
          await copy(
            data.map((datum) => datum[dir].Merged.nodesStr).join("\n")
          )}
      >
        {sum(data.map((datum) => datum[dir].Merged.nodes.length))}
      </td>
    {/each}
    <!-- <td
      aria-label={
        ''
      // [
      //   data[i].up.Merged.nodesStr,
      //   data[i].same.Merged.nodesStr,
      //   data[i].down.Merged.nodesStr,
      // ].join("\n")
      // }
      }
      on:click={async () =>
        await copy(
          [
            data[i].up.Merged.nodesStr,
            data[i].same.Merged.nodesStr,
            data[i].down.Merged.nodesStr,
          ].join("\n")
        )}
    >
      {[
        ...data[i].up.Merged.nodes,
        ...data[i].same.Merged.nodes,
        ...data[i].down.Merged.nodes,
      ].length}
    </td> -->
  </tr>
  <tr>
    <td>Real Edges</td>
    {#each ["up", "same", "down"] as dir}
      <td
        aria-label={data.map((datum) => datum[dir].Merged.edgesStr).join("\n")}
        on:click={async () =>
          await copy(
            data.map((datum) => datum[dir].Merged.edgesStr).join("\n")
          )}
      >
        {sum(data.map((datum) => datum[dir].Merged.edges.length))}
      </td>
    {/each}
    <!-- <td
      aria-label={[
        data[i].up.Merged.edgesStr,
        data[i].same.Merged.edgesStr,
        data[i].down.Merged.edgesStr,
      ].join("\n")}
      on:click={async () =>
        await copy(
          [
            data[i].up.Merged.edgesStr,
            data[i].same.Merged.edgesStr,
            data[i].down.Merged.edgesStr,
          ].join("\n")
        )}
    >
      {[
        ...data[i].up.Merged.edges,
        ...data[i].same.Merged.edges,
        ...data[i].down.Merged.edges,
      ].length}
    </td> -->
  </tr>
  <tr>
    <td>Implied Edges</td>
    {#each ["up", "same", "down"] as dir}
      <td
        aria-label={data.map((datum) => datum[dir].Implied.edgesStr).join("\n")}
        on:click={async () =>
          await copy(
            data.map((datum) => datum[dir].Implied.edgesStr).join("\n")
          )}
      >
        {sum(data.map((datum) => datum[dir].Implied.edges.length))}
      </td>
    {/each}
    <!-- <td
      aria-label={[
        data[i].up.Implied.edgesStr,
        data[i].same.Implied.edgesStr,
        data[i].down.Implied.edgesStr,
      ].join("\n")}
      on:click={async () =>
        await copy(
          [
            data[i].up.Implied.edgesStr,
            data[i].same.Implied.edgesStr,
            data[i].down.Implied.edgesStr,
          ].join("\n")
        )}
    >
      {[
        ...data[i].up.Implied.edges,
        ...data[i].same.Implied.edges,
        ...data[i].down.Implied.edges,
      ].length}
    </td> -->
  </tr>
</table>

<!-- 
<table>
  <thead>
    <tr>
      <th scope="col">Measure</th>
      <th scope="col">Parent</th>
      <th scope="col">Sibling</th>
      <th scope="col">Child</th>
      <th scope="col">Total</th>
    </tr>
  </thead>

  <tr>
    <td><strong>Nodes</strong></td>
    <td data-tooltip={pRNodesStr} on:click={async () => await copy(pRNodesStr)}
      >{pRNodes.length}</td
    >
    <td data-tooltip={sRNodesStr} on:click={async () => await copy(sRNodesStr)}
      >{sRNodes.length}</td
    >
    <td data-tooltip={cRNodesStr} on:click={async () => await copy(cRNodesStr)}
      >{cRNodes.length}</td
    >
    <td>{pRNodes.length + sRNodes.length + cRNodes.length}</td>
  </tr>

  <tr>
    <td><strong>Real Edges</strong></td>
    <td data-tooltip={pREdgesStr} on:click={async () => await copy(pREdgesStr)}
      >{pREdges.length}</td
    >
    <td data-tooltip={sREdgesStr} on:click={async () => await copy(sREdgesStr)}
      >{sREdges.length}</td
    >
    <td data-tooltip={cREdgesStr} on:click={async () => await copy(cREdgesStr)}
      >{cREdges.length}</td
    >
    <td>{pREdges.length + sREdges.length + cREdges.length}</td>
  </tr>

  <tr>
    <td><strong>Implied Edges</strong></td>
    <td data-tooltip={pIEdgesStr} on:click={async () => await copy(pIEdgesStr)}
      >{pAEdges.length - pREdges.length}</td
    >
    <td data-tooltip={sIEdgesStr} on:click={async () => await copy(sIEdgesStr)}
      >{sAEdges.length - sREdges.length}</td
    >
    <td data-tooltip={cIEdgesStr} on:click={async () => await copy(cIEdgesStr)}
      >{cAEdges.length - cREdges.length}</td
    >
    <td>{pREdges.length + sREdges.length + cREdges.length}</td>
  </tr>
</table> -->
<style>
  table {
    border-collapse: collapse;
  }

  td:first-child {
    text-align: right;
  }
  td,
  th {
    padding: 3px;
    border: 1px solid var(--background-modifier-border);
    white-space: pre-line;
  }

  td.filler {
    opacity: 0;
  }
</style>
