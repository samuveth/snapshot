<template>
  <UiCollapsible
    :hideRemove="config.preview"
    :number="index + 1"
    :open="open"
    :title="title"
    @remove="$emit('remove')"
    @toggle="open = !open"
  >
    <UiSelect
      :disabled="config.preview"
      :modelValue="type"
      @update:modelValue="handleTypeChange($event)"
    >
      <template v-slot:label>type</template>
      <option value="transferFunds">Transfer Funds</option>
      <option value="transferNFT">Transfer NFT</option>
      <option value="contractInteraction">Contract Interaction</option>
      <option value="raw">Raw Transaction</option>
    </UiSelect>

    <PluginSafeSnapFormContractInteraction
      v-if="type === 'contractInteraction'"
      :config="config"
      :modelValue="modelValue"
      :nonce="nonce"
      @update:modelValue="$emit('update:modelValue', $event)"
    />

    <PluginSafeSnapFormTransferFunds
      v-if="type === 'transferFunds'"
      :config="config"
      :modelValue="modelValue"
      :nonce="nonce"
      @update:modelValue="$emit('update:modelValue', $event)"
    />

    <PluginSafeSnapFormSendAsset
      v-if="type === 'transferNFT'"
      :config="config"
      :modelValue="modelValue"
      :nonce="nonce"
      @update:modelValue="$emit('update:modelValue', $event)"
    />

    <PluginSafeSnapFormRawTransaction
      v-if="type === 'raw'"
      :modelValue="modelValue"
      :nonce="nonce"
      :config="config"
      @update:modelValue="$emit('update:modelValue', $event)"
    />
  </UiCollapsible>
</template>

<script>
import { formatUnits } from '@ethersproject/units';
import { getAbiFirstFunctionName } from '@/helpers/abi/utils';

const labels = {
  contractInteraction: 'Contract Interaction',
  transferFunds: 'Transfer Funds',
  transferNFT: 'Transfer NFT',
  raw: 'Raw Transaction'
};
export default {
  props: ['modelValue', 'index', 'nonce', 'config'],
  emits: ['update:modelValue', 'remove'],
  data() {
    let type = 'transferFunds';
    if (this.modelValue) {
      type = this.modelValue.type ? this.modelValue.type : 'raw';
    }

    return {
      open: !this.config.preview,
      type
    };
  },
  watch: {
    modelValue() {
      if (this.modelValue?.type) {
        this.type = this.modelValue.type;
      }
    }
  },
  mounted() {
    if (!this.config.preview) this.$emit('update:modelValue', undefined);
    if (this.config.preview && !this.modelValue.type) {
      this.type = 'raw';
    }
  },
  computed: {
    title() {
      if (this.modelValue) {
        try {
          const addr = this._shorten(this.modelValue.to);
          const type = this.modelValue.type || this.type;
          switch (type) {
            case 'contractInteraction':
              return `${getAbiFirstFunctionName(this.modelValue.abi)}() - ${
                this.modelValue.value
              } wei to ${addr}`;
            case 'transferFunds':
              return `Transfer ${formatUnits(
                this.modelValue.amount,
                this.modelValue.token.decimals
              )} ${this.modelValue.token.symbol} to ${addr}`;
            case 'transferNFT':
              return `Send ${this.modelValue.collectable.name} #${this._shorten(
                this.modelValue.collectable.id,
                10
              )} to ${addr}`;
            case 'raw':
              return `Send ${this.modelValue.value} wei to ${addr}`;
          }
        } catch (error) {
          console.log('could not determine title', error);
        }
      }
      return this.getLabel(this.type);
    }
  },
  methods: {
    getLabel(type) {
      return labels[type];
    },
    handleTypeChange(type) {
      this.type = type;
      this.$emit('update:modelValue', undefined);
    }
  }
};
</script>
