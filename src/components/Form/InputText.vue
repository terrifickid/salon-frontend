<template>
  <div>
    <FormLabel :count="count" :required="required">{{ title }}</FormLabel>

    <input
      type="text"
      class="font-haffer sm:ml-0 bg-transparent block border-b border-black w-full text-black py-3 outline-none placeholder-opb"
      placeholder=""
      maxlength="250"
      required
      v-model="value"
      @input="$emit('update', value)"
      @keyup.enter="$emit('ready')"
      autocomplete="off"
    />
    <FormHelp :help="help" v-if="help" />
    <FormButtonOk @ready="$emit('ready')" @back="$emit('back')" />
  </div>
</template>
<script>
import FormLabel from "@/components/Form/FormLabel.vue";
import FormHelp from "@/components/Form/FormHelp.vue";
import FormButtonOk from "@/components/Form/FormButtonOk.vue";
export default {
  components: {
    FormLabel,
    FormButtonOk,
    FormHelp,
  },
  props: ["count", "title", "required", "placeholder", "help", "id", "cached"],
  emits: ["ready"],
  data() {
    return {
      value: "",
    };
  },
  mounted() {
    if (this.cached === true && localStorage.getItem(this.id) !== null) {
      this.value = localStorage.getItem(this.id);
    }
  },
};
</script>
