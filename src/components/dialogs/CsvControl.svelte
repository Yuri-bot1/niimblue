<script lang="ts">
  import { tr } from "$/utils/i18n";
  import { dsvFormat } from "d3-dsv";
  import { type CsvParams } from "$/types";
  import { csvData } from "$/stores/app";
  import { Toasts } from "$/utils/toasts";

  interface Props {
    enabled: boolean;
    onPlaceholderPicked: (name: string) => void;
  }

  let { enabled = $bindable(), onPlaceholderPicked }: Props = $props();

  let placeholders = $state<string[]>([]);
  let rows = $state<number>(0);
  let dragOver = $state<boolean>(false);

  const parse = (csv: CsvParams) => {
    const parser = dsvFormat(csv.separator);
    const result = parser.parse(csv.data ? `${csv.headers}\n${csv.data}` : csv.headers);
    placeholders = result.columns;
    rows = result.length;
  };

  const changeSeparator = (separator: CsvParams["separator"]) => {
    const currentSeparator = $csvData.separator;
    const header = dsvFormat(currentSeparator).parseRows($csvData.headers)[0] ?? [];
    csvData.update((current) => ({
      ...current,
      separator,
      headers: dsvFormat(separator).formatRows([header]),
    }));
  };

  const onCsvDrop = async (event: DragEvent) => {
    event.preventDefault();
    dragOver = false;
    const file = event.dataTransfer?.files[0];
    if (!file) return;

    if (!/\.(csv|txt)$/i.test(file.name)) {
      Toasts.error($tr("params.csv.filetype"));
      return;
    }

    try {
      $csvData.data = await file.text();
      enabled = true;
    } catch (error) {
      Toasts.error(error);
    }
  };

  $effect(() => {
    parse($csvData);
  });
</script>

<section class="csv-panel card">
  <div class="card-header d-flex flex-wrap justify-content-between align-items-center gap-2">
    <h2 class="fs-6 mb-0">{$tr("params.csv.title")}</h2>
    <div class="form-check form-switch mb-0">
      <input class="form-check-input" type="checkbox" role="switch" id="csv-enabled" bind:checked={enabled} />
      <label class="form-check-label" for="csv-enabled">{$tr("params.csv.enabled")}</label>
    </div>
  </div>
  <div class="card-body">
    <div class="mb-2">{$tr("params.csv.tip")}</div>
    <div class="mb-3">
      <label class="form-label mb-1" for="csv-headers">{$tr("params.csv.headers")}</label>
      <div class="input-group">
        <input id="csv-headers" class="form-control" bind:value={$csvData.headers} oninput={() => (enabled = true)} />
        <select
          class="form-select separator-select"
          aria-label={$tr("params.csv.separator")}
          value={$csvData.separator}
          onchange={(event) => changeSeparator(event.currentTarget.value as CsvParams["separator"])}>
          <option value=",">,</option>
          <option value=";">;</option>
        </select>
      </div>
    </div>
    <label class="form-label mb-1" for="csv-data">{$tr("params.csv.data")}</label>
    <textarea
      id="csv-data"
      class="dsv form-control"
      class:drag-over={dragOver}
      aria-label={$tr("params.csv.data")}
      bind:value={$csvData.data}
      oninput={() => (enabled = true)}
      ondragover={(event) => {
        event.preventDefault();
        dragOver = true;
      }}
      ondragleave={() => (dragOver = false)}
      ondrop={onCsvDrop}></textarea>

    <div class="placeholders pt-2">
      {$tr("params.csv.rowsfound")} <strong>{rows}</strong>
    </div>
    <div class="placeholders pt-1">
      {$tr("params.csv.placeholders")}
      {#each placeholders as p (p)}
        <button class="btn btn-sm btn-outline-info px-1 py-0" onclick={() => onPlaceholderPicked(p)}>{`{${p}}`}</button>
      {/each}
    </div>
  </div>
</section>

<style>
  .csv-panel {
    height: 100%;
  }

  .separator-select {
    flex: 0 0 5rem;
  }

  textarea.dsv {
    font-family: monospace;
    min-height: 22vh;
    resize: vertical;
  }

  textarea.dsv.drag-over {
    border-color: var(--bs-primary);
    box-shadow: 0 0 0 0.25rem rgba(var(--bs-primary-rgb), 0.25);
  }
</style>
