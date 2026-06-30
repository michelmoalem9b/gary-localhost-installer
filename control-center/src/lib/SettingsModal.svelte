<script lang="ts">
  import { invoke } from "@tauri-apps/api/core";
  import { open as openDialog } from "@tauri-apps/plugin-dialog";

  let {
    open = false,
    modelsDir,
    appsDir,
    onClose,
    onSave,
  }: {
    open: boolean;
    modelsDir: string | null;
    appsDir: string | null;
    onClose: () => void;
    onSave: (modelsDir: string | null, appsDir: string | null) => Promise<void>;
  } = $props();

  let draftModelsDir = $state(modelsDir ?? "");
  let draftAppsDir = $state(appsDir ?? "");
  let saving = $state(false);

  $effect(() => {
    if (open) {
      draftModelsDir = modelsDir ?? "";
      draftAppsDir = appsDir ?? "";
    }
  });

  async function browseModels() {
    const selected = await openDialog({
      directory: true,
      multiple: false,
    });
    if (selected && typeof selected === "string") {
      draftModelsDir = selected;
    }
  }

  async function browseApps() {
    const selected = await openDialog({
      directory: true,
      multiple: false,
    });
    if (selected && typeof selected === "string") {
      draftAppsDir = selected;
    }
  }

  async function handleSave() {
    saving = true;
    try {
      await onSave(
        draftModelsDir.trim() ? draftModelsDir : null,
        draftAppsDir.trim() ? draftAppsDir : null
      );
      onClose();
    } finally {
      saving = false;
    }
  }

  async function manageCustomApps() {
    try {
      await invoke("open_custom_services_config");
    } catch (e) {
      console.error("Failed to open custom apps config:", e);
    }
  }
</script>

{#if open}
  <div class="overlay">
    <button type="button" class="backdrop" aria-label="close settings" onclick={onClose}></button>
    <div class="modal" role="dialog" aria-modal="true" tabindex="-1">
      <div class="header">
        <div class="title">settings</div>
        <button class="close-btn" aria-label="close" onclick={onClose}>&times;</button>
      </div>

      <div class="body">
        <div class="field">
          <label for="models-dir">Models Directory</label>
          <div class="input-row">
            <input
              id="models-dir"
              type="text"
              bind:value={draftModelsDir}
              placeholder="Default: %APPDATA%/Gary4JUCE/models"
              disabled={saving}
            />
            <button type="button" onclick={browseModels} disabled={saving}>Browse</button>
          </div>
          <div class="help-text">
            Centralized folder for downloading and loading models.
          </div>
        </div>

        <div class="field">
          <label for="apps-dir">Apps Directory</label>
          <div class="input-row">
            <input
              id="apps-dir"
              type="text"
              bind:value={draftAppsDir}
              placeholder="Default: Repo services folder"
              disabled={saving}
            />
            <button type="button" onclick={browseApps} disabled={saving}>Browse</button>
          </div>
          <div class="help-text">
            Folder containing the bundled services (e.g. gary, carey).
          </div>
        </div>

        <div class="field custom-apps">
          <label>Custom Apps</label>
          <div class="help-text" style="margin-bottom: 8px;">
            Add your own apps like ComfyUI or acestep.cpp to the launcher by editing the custom config file.
          </div>
          <button type="button" onclick={manageCustomApps} disabled={saving}>
            Manage custom_services.json
          </button>
        </div>
      </div>

      <div class="actions">
        <button type="button" onclick={onClose} disabled={saving}>Cancel</button>
        <button type="button" class="accent" onclick={handleSave} disabled={saving}>Save</button>
      </div>
    </div>
  </div>
{/if}

<style>
  .overlay {
    position: fixed;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 24px;
    z-index: 100;
  }

  .backdrop {
    position: absolute;
    inset: 0;
    border: none;
    background: rgba(0, 0, 0, 0.7);
    padding: 0;
  }

  .modal {
    position: relative;
    z-index: 1;
    width: min(500px, 100%);
    border: 1px solid var(--border);
    background: var(--bg-secondary);
    box-shadow: 0 18px 50px rgba(0, 0, 0, 0.45);
    display: flex;
    flex-direction: column;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 18px;
    border-bottom: 1px solid var(--border);
  }

  .title {
    font-size: 14px;
    font-weight: 600;
    color: var(--text-primary);
  }

  .close-btn {
    background: none;
    border: none;
    color: var(--text-secondary);
    font-size: 20px;
    cursor: pointer;
    padding: 0;
    line-height: 1;
  }

  .close-btn:hover {
    color: var(--text-primary);
  }

  .body {
    padding: 18px;
    display: flex;
    flex-direction: column;
    gap: 20px;
  }

  .field {
    display: flex;
    flex-direction: column;
    gap: 6px;
  }

  .field label {
    font-size: 12px;
    font-weight: 600;
    color: var(--text-primary);
  }

  .input-row {
    display: flex;
    gap: 8px;
  }

  .input-row input {
    flex: 1;
    min-width: 0;
  }

  .help-text {
    font-size: 11px;
    color: var(--text-secondary);
    line-height: 1.4;
  }

  .custom-apps {
    padding-top: 16px;
    border-top: 1px solid var(--border-subtle);
  }

  .actions {
    padding: 12px 18px;
    display: flex;
    justify-content: flex-end;
    gap: 8px;
    border-top: 1px solid var(--border);
    background: var(--bg-tertiary);
  }
</style>
