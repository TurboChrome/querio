<template>
  <Button v-tooltip.bottom="{ content }" @click.stop="onExport">
    <FileExportIcon class="h-4 w-4" />
  </Button>
</template>

<script>
import Button from '@/components/base/Button.vue'

import FileExportIcon from '@/assets/file-export.svg'
import FileSaver from 'file-saver'
import {isPromise} from "@/utils";
import dayjs from "dayjs";
export default {
  components: {
    Button,
    FileExportIcon
  },
  props: {
    value: {
      required: true
    },
    name: {
      required: true
    }
  },
  data() {
    return {
      exported: false
    }
  },
  computed: {
    content() {
      return this.exported ? 'Exported!' : 'Export'
    }
  },
  methods: {
    async onExport() {
      const fileContent = isPromise(this.value) ? await this.value : this.value
      const blob = new Blob([fileContent], {type: "text/plain;charset=utf-8"});
      FileSaver.saveAs(blob, `${dayjs().format('YYYYMMDDHHmmss')}_${this.name}.txt`);
    }
  }
}
</script>