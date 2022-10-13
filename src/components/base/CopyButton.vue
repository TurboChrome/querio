<template>
  <Button v-tooltip.bottom="{ content }" @click.stop="onCopy">
    <CopyIcon class="h-4 w-4" />
  </Button>
</template>

<script>
import Button from '@/components/base/Button.vue'

import CopyIcon from '@/assets/copy.svg'
import {isPromise} from "@/utils";

export default {
  components: {
    Button,
    CopyIcon
  },
  props: {
    value: {
      required: true
    }
  },
  data() {
    return {
      copied: false
    }
  },
  computed: {
    content() {
      return this.copied ? 'Copied!' : 'Copy'
    }
  },
  methods: {
    async onCopy() {
      this.$copyText(isPromise(this.value) ? await this.value : this.value).then(() => {
        this.copied = true
        setTimeout(() => this.copied = false, 1500)
      })
    }
  }
}
</script>