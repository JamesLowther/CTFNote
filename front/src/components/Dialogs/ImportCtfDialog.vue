<template>
  <q-dialog ref="dialogRef" @hide="onDialogHide">
    <q-card style="min-width: 400px">
      <q-card-section class="text-h6"> Import CTF from CTFtime </q-card-section>
      <q-separator />
      <q-card-section>
        <q-input
          v-model="model"
          filled
          label="Id"
          hint="ctftime.org url or id"
          :rules="[validate]"
          autofocus
          @keypress.enter="submit"
        />
      </q-card-section>
      <q-card-actions align="right" class="q-gutter-md q-px-md q-pb-sm">
        <q-btn color="warning" flat label="cancel" @click="onDialogCancel" />
        <q-btn color="positive" label="import" @click="submit" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script lang="ts">
import { useDialogPluginComponent } from 'quasar';
import ctfnote from 'src/ctfnote';
import { defineComponent, ref } from 'vue';

function parseCtftimeId(s: string): number | null {
  const url = s.trim();
  const idReg = /^\d+$/;
  const urlReg = /^(?:https?:\/\/)?ctftime\.org\/event\/(\d+)(?:\/.*)?$/i;
  if (idReg.exec(url)) {
    return parseInt(url);
  }
  const match = urlReg.exec(url);
  if (!match) return null;
  return parseInt(match[1]);
}

export default defineComponent({
  emits: useDialogPluginComponent.emits,
  setup() {
    const { dialogRef, onDialogHide, onDialogOK, onDialogCancel } =
      useDialogPluginComponent();

    return {
      wrapNotify: ctfnote.ui.useWrapNotify(),
      importCtf: ctfnote.ctfs.useImportCtf(),
      dialogRef,
      model: ref(''),
      onDialogHide,
      onDialogOK,
      onDialogCancel,
    };
  },
  methods: {
    async submit() {
      const id = parseCtftimeId(this.model);
      if (id === null) return;
      const success = await this.wrapNotify(() => this.importCtf(id));
      if (success) {
        this.onDialogOK();
      }
    },
    validate() {
      if (this.model && parseCtftimeId(this.model) === null)
        return 'Invalid url or id';
    },
  },
});
</script>

<style scoped></style>
