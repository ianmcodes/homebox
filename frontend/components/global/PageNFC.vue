<script setup lang="ts">
  import { useI18n } from "vue-i18n";
  import { DialogID } from "@/components/ui/dialog-provider/utils";
  import { Dialog, DialogContent, DialogHeader, DialogTitle } from "@/components/ui/dialog";
  import MdiNfcVariant from "~icons/mdi/nfc-variant";
  import { Button } from "@/components/ui/button";
  import { Tooltip, TooltipContent, TooltipTrigger } from "@/components/ui/tooltip";
  import { useDialog } from "@/components/ui/dialog-provider";
  import { toast } from "@/components/ui/sonner";

  interface NDEFReaderInstance {
    write(
      message: { records: { recordType: string; data: string }[] },
      options?: { signal?: AbortSignal }
    ): Promise<void>;
  }

  interface NDEFReaderConstructor {
    new (): NDEFReaderInstance;
  }

  const { t } = useI18n();
  const { openDialog, closeDialog } = useDialog();
  const nfcSupported = "NDEFReader" in window;

  function handleNFCClick() {
    if ("NDEFReader" in window) {
      const ndef = new (window as Window & { NDEFReader: NDEFReaderConstructor }).NDEFReader();
      const controller = new AbortController();
      openDialog(DialogID.PageNFC, {
        onClose: () => {
          if (controller) {
            controller.abort("Dialog closed");
          }
        },
      });
      ndef
        .write({ records: [{ recordType: "url", data: window.location.href }] }, { signal: controller.signal })
        .then(() => {
          toast.success(t("components.global.page_nfc_tag.nfc_writing_success"));
        })
        .catch((error: unknown) => {
          if (error === "Dialog closed") {
            // NFC writing was aborted by closing the dialog
            return;
          }
          console.error("Error writing to NFC tag:", error);
          toast.error(t("components.global.page_nfc_tag.nfc_writing_not_supported"));
        })
        .finally(() => {
          closeDialog(DialogID.PageNFC);
        });
    } else {
      console.warn("NFC is not supported on this device.");
    }
  }
</script>

<template>
  <Dialog :dialog-id="DialogID.PageNFC">
    <DialogContent>
      <DialogHeader>
        <DialogTitle>
          {{ $t("components.global.page_nfc_tag.title") }}
        </DialogTitle>
      </DialogHeader>
      <div>
        <p>{{ $t("components.global.page_nfc_tag.approach_nfc_tag") }}</p>
        <p>
          <strong>{{ $t("components.global.page_nfc_tag.warning") }}</strong>
        </p>
      </div>
    </DialogContent>
  </Dialog>

  <Tooltip v-if="nfcSupported">
    <TooltipTrigger as-child>
      <Button size="icon" @click="handleNFCClick()">
        <MdiNfcVariant name="mdi-nfc-variant" />
      </Button>
    </TooltipTrigger>
    <TooltipContent>
      {{ $t("components.global.page_nfc_tag.nfc_tooltip") }}
    </TooltipContent>
  </Tooltip>
</template>
