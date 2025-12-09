<template>
  <div>
    <FormLabel :count="count" :required="required">{{ title }}</FormLabel>

    <textarea
      class="font-haffer sm:ml-0 bg-transparent block border border-black w-full text-black p-5 outline-none placeholder-opb"
      placeholder=""
      maxlength="50000"
      required
      rows="3"
      v-model="value"
      @input="$emit('update', value)"
      autocomplete="off"
    />
    <!-- Character counter -->
    <div class="flex justify-end mt-2 text-sm text-gray-600">
      {{ value.length }} / 50000
    </div>
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
  props: ["count", "title", "required", "placeholder", "help"],
  emits: ["ready"],
  data() {
    return {
      value: "",
    };
  },
};
</script>
